# 🍉 온라인 멀티플레이 수박게임 (Fruit Game Battle)

> 실시간 온라인 대전이 가능한 수박게임(Suika Game) - 친구들과 함께 즐기는 과일 합체 배틀!

[![Development Status](https://img.shields.io/badge/status-in%20development-yellow)](https://github.com/cldhfleks2/FruitGameBattle)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)
[![Java Version](https://img.shields.io/badge/java-17-blue)](https://www.oracle.com/java/)
[![Spring Boot](https://img.shields.io/badge/springboot-3.2.0-green)](https://spring.io/projects/spring-boot)
[![Vue.js](https://img.shields.io/badge/vue.js-3.3.4-brightgreen)](https://vuejs.org/)

## 📋 프로젝트 개요

전통적인 수박게임을 **실시간 온라인 멀티플레이**로 확장한 웹 기반 게임입니다. Spring Boot WebSocket(STOMP)를 활용한 실시간 통신과 Matter.js 물리엔진을 통해 부드러운 게임플레이를 제공합니다.

### 🎮 주요 특징
- **실시간 멀티플레이**: 전 세계 플레이어와 실시간 대전
- **다양한 게임 모드**: 시간제한, 턴제, 배틀 모드
- **분할 화면**: 상대방 플레이 실시간 관전
- **공격/방어 시스템**: 전략적 플레이 요소

## 🚀 개발 현황

### Phase 1: 기본 멀티플레이 
- [x] 싱글플레이 수박게임 구현 (Matter.js)
- [ ] Spring WebSocket(STOMP) 실시간 통신 구축
- [ ] 양쪽 화면 분할 UI
- [ ] 기본 매칭 시스템

### Phase 2: 게임 모드 구현 
- [ ] 시간 제한 대결 모드
- [ ] 턴제 모드
- [ ] 배틀 모드 (방해 시스템)
- [ ] 모드별 규칙 및 승리 조건

### Phase 3: 고도화
- [ ] 매칭메이킹 시스템
- [ ] 리더보드
- [ ] 리플레이 기능
- [ ] 관전 모드

### Phase 4: 부가 기능
- [ ] 커스텀 과일 스킨
- [ ] 토너먼트 시스템
- [ ] 플레이어 프로필 및 통계
- [ ] AI 유전 알고리즘 테스트

## 🛠️ 기술 스택

### Backend
| 기술                   | 버전    | 용도           |
|----------------------|-------|--------------|
| **Spring Boot**      | 3.2.0 | 백앤드 메인 프레임워크 |
| **Java**             | 17    | 백엔드 개발 언어    |
| **Spring WebSocket** | -     | 실시간 양방향 통신   |
| **MyBatis**          | 3.0.3 | 메타 데이터 저장    |
| **MongoDB**          | 8.0   | 로그 저장        |
| **Spring Security**  | -     | 사용자 인증       |
| **Lombok**           | -     | 편의           |

### Frontend
| 기술                 | 버전     | 용도              |
|--------------------|--------|-----------------|
| **Vue.js**         | 3.3.4  | 프론트엔드 프레임워크     |
| **Vite**           | 4.4.9  | 빌드 도구           |
| **Matter.js**      | 0.19.0 | 2D 물리 엔진        |
| **@stomp/stompjs** | 7.0.0  | WebSocket 클라이언트 |
| **SockJS Client**  | 1.6.1  | WebSocket 폴백    |
| **Vue Router**     | 4.2.4  | 라우팅             |
| **Axios**          | 1.5.0  | HTTP 클라이언트      |
| **Sass**           | 1.66.1 | CSS 전처리기        |

### 선택적 추가 기술
| 기술         | 용도              |
|------------|-----------------|
| **Docker** | 컨테이너화 배포        |

## 📦 설치 및 실행

### 필수 요구사항
- Java 17 이상
- Node.js 16.0.0 이상
- Maven 3.8 이상
- MariaDB 10.6 이상 (메타데이터)
- MongoDB 7.0 이상 (로그 저장)
- npm 또는 yarn

### Backend 설치 및 실행

```bash
```

### Frontend 설치 및 실행

```bash
```

### 환경 변수 설정

#### Backend (application.yml)


#### Frontend (vite.config.js)


## 🎮 게임 모드

### 1. 베이직 모드 🎯
- 각자 독립된 게임판에서 플레이
- 실시간으로 게임 플레이 및 상대방 플레이 관전 가능
- 게임 오버 시 개별 재시작 가능
- 시간 제한 없이 최고 점수 도전

### 2. 시간 제한 대결 ⏱️
- 3분 제한 시간 내 최고 점수 달성
- 독립된 게임판에서 플레이
- 안정적 플레이 vs 고득점 도전의 전략적 선택

### 3. 턴제 협동/경쟁 🔄
- 하나의 게임판을 번갈아가며 플레이
- 턴당 10초 제한
- 협동과 경쟁의 묘한 조합

### 4. 배틀 모드 💥
- 특정 과일 합체 시 상대방에게 방해 요소 전송
- 방어막과 카운터 시스템
- 테트리스 배틀과 유사한 긴장감

## 🏗️ 프로젝트 구조


## 🤝 개발 로그

개발 계획과 진행 상황은 [DEVELOPMENT.md](DEVELOPMENT.md)를 참고해주세요.

## 📝 라이선스

이 프로젝트는 MIT 라이선스 하에 배포됩니다. 자세한 내용은 [LICENSE](LICENSE) 파일을 참고하세요.

## 📞 문의

- **이메일**: cldhfleks2@naver.com
- **Issues**: [GitHub Issues](https://github.com/cldhfleks2/FruitGameBattle/issues)

## 🙏 감사의 말

- [Matter.js](https://brm.io/matter-js/) - 2D 물리 엔진
- [Spring Boot](https://spring.io/projects/spring-boot) - Java 웹 프레임워크
- [Vue.js](https://vuejs.org/) - 프론트엔드 프레임워크
- [STOMP](https://stomp.github.io/) - 메시징 프로토콜
- 원작 수박게임 개발자분들께 감사드립니다

---

⭐ 이 프로젝트가 마음에 드신다면 Star를 눌러주세요!