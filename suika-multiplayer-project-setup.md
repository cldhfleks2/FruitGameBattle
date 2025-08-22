# 수박게임 멀티플레이 프로젝트 세팅 가이드

## 1. 백엔드 (Spring Boot)

### pom.xml 의존성
```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- WebSocket -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-websocket</artifactId>
    </dependency>
    
    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>3.0.3</version>
    </dependency>
    
    <!-- MySQL or H2 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <!-- Lombok (선택) -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    
    <!-- JWT (유저 인증용) -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
</dependencies>
```

### WebSocket 설정
```java
// WebSocketConfig.java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        config.enableSimpleBroker("/topic", "/queue");
        config.setApplicationDestinationPrefixes("/app");
    }
    
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws-game")
                .setAllowedOrigins("http://localhost:5173")
                .withSockJS();
    }
}
```

### 게임 컨트롤러
```java
// GameController.java
@Controller
@RequiredArgsConstructor
public class GameController {
    
    private final SimpMessagingTemplate messagingTemplate;
    
    @MessageMapping("/game/move")
    @SendTo("/topic/game/{roomId}")
    public GameState handleMove(MoveData moveData) {
        // 게임 로직 처리
        return gameState;
    }
    
    @MessageMapping("/game/attack")
    public void handleAttack(AttackData attack, @DestinationVariable String roomId) {
        messagingTemplate.convertAndSend("/topic/game/" + roomId + "/attack", attack);
    }
}
```

## 2. 프론트엔드 (Vue.js + Vite)

### package.json
```json
{
  "name": "suika-multiplayer-client",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "vue": "^3.3.4",
    "vue-router": "^4.2.4",
    "pinia": "^2.1.6",
    "matter-js": "^0.19.0",
    "@stomp/stompjs": "^7.0.0",
    "sockjs-client": "^1.6.1",
    "axios": "^1.5.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.3.4",
    "vite": "^4.4.9",
    "sass": "^1.66.1"
  }
}
```

### Vite 설정
```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'

export default defineConfig({
  plugins: [vue()],
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      },
      '/ws-game': {
        target: 'ws://localhost:8080',
        ws: true,
        changeOrigin: true
      }
    }
  }
})
```

### WebSocket 연결 클래스
```javascript
// services/GameWebSocket.js
import { Client } from '@stomp/stompjs';
import SockJS from 'sockjs-client';

export class GameWebSocket {
  constructor() {
    this.client = null;
    this.roomId = null;
  }
  
  connect(roomId, onMessageCallback) {
    this.roomId = roomId;
    
    this.client = new Client({
      webSocketFactory: () => new SockJS('http://localhost:8080/ws-game'),
      onConnect: () => {
        // 게임 상태 구독
        this.client.subscribe(`/topic/game/${roomId}`, (message) => {
          const gameState = JSON.parse(message.body);
          onMessageCallback('gameState', gameState);
        });
        
        // 공격 이벤트 구독
        this.client.subscribe(`/topic/game/${roomId}/attack`, (message) => {
          const attack = JSON.parse(message.body);
          onMessageCallback('attack', attack);
        });
      }
    });
    
    this.client.activate();
  }
  
  sendMove(moveData) {
    this.client.publish({
      destination: '/app/game/move',
      body: JSON.stringify(moveData)
    });
  }
  
  sendAttack(attackData) {
    this.client.publish({
      destination: `/app/game/attack/${this.roomId}`,
      body: JSON.stringify(attackData)
    });
  }
}
```

### Vue 게임 컴포넌트
```vue
<!-- components/GameBoard.vue -->
<template>
  <div class="game-container">
    <div class="player-area">
      <canvas ref="myCanvas"></canvas>
      <div class="score">{{ myScore }}</div>
    </div>
    
    <div class="opponent-area">
      <canvas ref="opponentCanvas"></canvas>
      <div class="score">{{ opponentScore }}</div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import Matter from 'matter-js'
import { GameWebSocket } from '@/services/GameWebSocket'

const myCanvas = ref(null)
const opponentCanvas = ref(null)
const myScore = ref(0)
const opponentScore = ref(0)

let engine, render, gameWS

onMounted(() => {
  // Matter.js 초기화
  engine = Matter.Engine.create()
  render = Matter.Render.create({
    canvas: myCanvas.value,
    engine: engine,
    options: {
      width: 400,
      height: 600,
      wireframes: false
    }
  })
  
  // WebSocket 연결
  gameWS = new GameWebSocket()
  gameWS.connect(roomId, handleGameMessage)
  
  // 게임 시작
  Matter.Render.run(render)
  Matter.Runner.run(engine)
})

function handleGameMessage(type, data) {
  if (type === 'gameState') {
    updateOpponentView(data)
  } else if (type === 'attack') {
    handleIncomingAttack(data)
  }
}

function dropFruit(x) {
  const fruit = Matter.Bodies.circle(x, 50, 30, {
    restitution: 0.3,
    friction: 0.1
  })
  
  Matter.World.add(engine.world, fruit)
  
  // 서버에 움직임 전송
  gameWS.sendMove({ x, fruitType: currentFruit })
}

onUnmounted(() => {
  Matter.Render.stop(render)
  Matter.Engine.clear(engine)
  gameWS.disconnect()
})
</script>
```

## 3. 데이터베이스 스키마 (MyBatis)

### SQL
```sql
-- 유저 테이블
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 게임 기록 테이블
CREATE TABLE game_records (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    player1_id BIGINT,
    player2_id BIGINT,
    game_mode VARCHAR(20),
    winner_id BIGINT,
    player1_score INT,
    player2_score INT,
    duration INT,
    played_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (player1_id) REFERENCES users(id),
    FOREIGN KEY (player2_id) REFERENCES users(id)
);

-- 랭킹 테이블
CREATE TABLE rankings (
    user_id BIGINT PRIMARY KEY,
    total_score BIGINT DEFAULT 0,
    wins INT DEFAULT 0,
    losses INT DEFAULT 0,
    highest_score INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### MyBatis Mapper
```xml
<!-- GameMapper.xml -->
<mapper namespace="com.suika.mapper.GameMapper">
    <insert id="saveGameRecord" parameterType="GameRecord">
        INSERT INTO game_records 
        (player1_id, player2_id, game_mode, winner_id, player1_score, player2_score, duration)
        VALUES 
        (#{player1Id}, #{player2Id}, #{gameMode}, #{winnerId}, #{player1Score}, #{player2Score}, #{duration})
    </insert>
    
    <select id="getRankings" resultType="RankingDto">
        SELECT u.username, r.total_score, r.wins, r.highest_score
        FROM rankings r
        JOIN users u ON r.user_id = u.id
        ORDER BY r.total_score DESC
        LIMIT 100
    </select>
</mapper>
```

## 4. 프로젝트 구조

```
suika-multiplayer/
├── backend/
│   ├── src/main/java/com/suika/
│   │   ├── config/
│   │   │   ├── WebSocketConfig.java
│   │   │   └── MyBatisConfig.java
│   │   ├── controller/
│   │   │   ├── GameController.java
│   │   │   └── UserController.java
│   │   ├── service/
│   │   │   ├── GameService.java
│   │   │   └── MatchmakingService.java
│   │   ├── mapper/
│   │   │   └── GameMapper.java
│   │   └── dto/
│   │       ├── GameState.java
│   │       └── MoveData.java
│   └── src/main/resources/
│       ├── mapper/
│       │   └── GameMapper.xml
│       └── application.yml
│
└── frontend/
    ├── src/
    │   ├── components/
    │   │   ├── GameBoard.vue
    │   │   ├── GameLobby.vue
    │   │   └── Matchmaking.vue
    │   ├── services/
    │   │   ├── GameWebSocket.js
    │   │   └── GameEngine.js
    │   ├── stores/
    │   │   └── gameStore.js
    │   └── App.vue
    ├── package.json
    └── vite.config.js
```

## 5. 추가 고려사항

### 필수 추가 사항
1. **CORS 설정** - Spring Boot에서 Vue.js 요청 허용
2. **세션 관리** - Spring Session 또는 JWT
3. **매칭 시스템** - Redis 추가 고려 (실시간 매칭 큐)

### 선택적 추가 사항
1. **Redis** - 게임 상태 캐싱, 매칭 큐
2. **Docker** - 배포 환경 구성
3. **Nginx** - 리버스 프록시, 로드 밸런싱

이 구성으로 충분히 멀티플레이 수박게임을 구현할 수 있습니다!