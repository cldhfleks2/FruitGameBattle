# 수박게임 멀티플레이 프로젝트 세팅 가이드

## 1. 백엔드 (Spring Boot)

### pom.xml 의존성

### WebSocket 설정

### package.json
```json
{
}
```

### Vite 설정

### WebSocket 연결 클래스


### Vue 게임 컴포넌트

## 3. 데이터베이스 스키마 (MyBatis)

### SQL
```sql
-- 유저 테이블
CREATE TABLE users (
);

-- 게임 기록 테이블
CREATE TABLE game_records (
);

-- 랭킹 테이블
CREATE TABLE rankings (
);
```

### MyBatis Mapper

## 4. 프로젝트 구조

## 5. 추가 고려사항

### 필수 추가 사항
1. **CORS 설정** - Spring Boot에서 Vue.js 요청 허용
2. **세션 관리** - Spring Security
3. **매칭 시스템** - Redis 추가 고려 (실시간 매칭 큐)

### 선택적 추가 사항
1. **Docker** - 배포 환경 구성