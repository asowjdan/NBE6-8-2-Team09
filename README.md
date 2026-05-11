# 💰 나의 기여 내역 — NBE6-8-2-Team09 (가상화폐 모의투자 시스템)

> 백엔드 초기 세팅부터 거래소(Exchange) 도메인 핵심 기능 구현을 전담하였으며,
> 업비트 REST API 연동 및 Redis 기반 실시간 데이터 수집 로직을 구현하였습니다.
> 프론트엔드 환경 세팅 및 차트/코인 목록 페이지 개발도 일부 담당하였습니다.

---

## 🛠 기술 스택

`Java` `Spring Boot` `JPA` `Redis` `WebSocket` `MySQL` `GitHub Actions (CI)`

---

## 📁 담당 작업 내역

### 1. 백엔드 프로젝트 초기 세팅
- Spring Boot 기반 백엔드 프로젝트 초기 구조 구성
- Gradle Kotlin DSL 빌드 스크립트 설정
- `@EnableScheduling` 적용 및 스케줄러 환경 구성

### 2. 거래소(Exchange) 도메인 구현
- `exchange` 도메인 전체 설계 및 구현 (entity, repository, service, controller, dto)
- 거래소-코인 매핑 기능 구현
- ExchangeService 로직 작성 및 단위 테스트 코드 작성
- `ExchangeBackupScheduler.java` — 스케줄러를 통한 주기적 거래소 데이터 백업 구현

### 3. 업비트 API + Redis 연동 (WebSocket 기반 실시간 수집)
- 업비트 REST API를 활용한 Redis 초기 캔들 데이터 적재 로직 구현
- `CandleWebSocketHandler.java` — 업비트 WebSocket 연결 후 실시간 캔들 데이터 수신 및 Redis 저장
- `UpbitRestCandleFetcher.java` — REST API 배치(200개 단위) 호출로 과거 캔들 데이터 수집
- `UpbitRestScheduler.java` — cron 기반 주기적 데이터 갱신 (30분/1시간/1일/1주 단위 분리)
- `MockCoinListProvider.java` — 지원 코인 목록 하드코딩 관리 컴포넌트 구현
- `RedisInitializer.java` — Spring 구동 완료 후 별도 스레드로 전체 인터벌 캔들 초기화 실행

### 4. CI 테스트 오류 수정
- GitHub Actions 기반 CI 파이프라인 테스트 오류 3차례 수정

### 5. 프론트엔드 작업 (일부)
- 프론트엔드 환경 세팅 (React + TypeScript)
- 목업 데이터를 활용한 차트 및 코인 목록 페이지 구현
- 프론트 오류 해결 및 페이지 수정

---

## 🚧 문제 상황과 해결

### 1. 업비트 API Rate Limit(429) 로 인한 초기화 중단
- **상황**: 서버 시작 시 업비트 REST API로 대량의 캔들 데이터(SEC/MIN/HOUR/DAY/WEEK 등 전체 인터벌)를 Redis에 적재하는 과정에서 `429 Too Many Requests` 에러가 발생하며 수집이 중단됐습니다.
- **해결**: API 호출 사이에 200ms 딜레이(`BETWEEN_CALL_DELAY_MS`)를 두고, 429 발생 시 3분 대기(`RATE_LIMIT_DELAY_MS`) 후 재시도하는 로직을 추가했습니다. 또한 초기화 작업 자체를 `new Thread(this::initializeCandles).start()`로 별도 스레드에서 실행해 Spring 구동 지연을 방지했습니다.
- **배운 점**: 외부 API 연동 시 Rate Limit 처리 전략과 애플리케이션 초기화 비동기화의 중요성을 체감했습니다.

### 2. CoinPriceResponse / ExchangeDTO 타입 불일치로 인한 데이터 매핑 오류
- **상황**: ExchangeService 테스트 중 가격 계산 결과가 틀리게 나왔습니다. 원인을 추적하니 `CoinPriceResponse.price`가 `String`으로 선언되어 있어 `AnalyticsService`에서 `new BigDecimal(price)` 변환 시 불필요한 형변환과 잠재적 파싱 오류가 발생했고, `ExchangeDTO`의 timestamp도 `long` 타입으로 선언되어 날짜 처리에 문제가 있었습니다.
- **해결**: `CoinPriceResponse.price`를 `BigDecimal`, `time`을 `LocalDateTime`으로, `ExchangeDTO`의 가격 필드 전체(`open`, `high`, `low`, `close`)를 `double` → `BigDecimal`로 변경하고 `AnalyticsService`의 불필요한 형변환 코드를 제거했습니다.
- **배운 점**: 금융 데이터는 처음부터 `BigDecimal`로 설계해야 하며, DTO 타입 설계가 서비스 계층 전체에 미치는 영향을 고려해야 한다는 것을 배웠습니다.

### 3. 스케줄러 설계 문제 — 단일 fixedDelay에서 cron 분리
- **상황**: 초기에 `@Scheduled(fixedDelay = 5분)`으로 모든 캔들 인터벌(MIN_1, MIN_30, HOUR_1 등)을 한 번에 수집하는 단일 메서드로 구현했습니다. 이 방식은 각 인터벌의 특성(30분봉은 30분마다, 일봉은 하루 한 번)을 무시하고 불필요한 API 호출이 발생했습니다.
- **해결**: `@Scheduled(cron = "0 0/30 * * * *")` 형태로 인터벌별 cron 표현식을 사용해 `fetchEvery30Min()`, `fetchHourly()`, `fetchDaily()`, `fetchWeekly()` 메서드로 분리했습니다.
- **배운 점**: 스케줄러 설계 시 데이터의 갱신 주기에 맞는 cron 전략이 API 비용과 시스템 효율에 직결된다는 것을 배웠습니다.

---

# 가상화폐 모의 투자 시스템
<br>




[![Back9-Home-orange.png](https://i.postimg.cc/vTbwcMt9/Back9-Home-orange.png)](https://postimg.cc/Pp3RRgcr)


[`가상회폐 모의투자 시스템 링크`](https://peuronteuendeu.onrender.com/)


<br>


## 개요
1. [소개](#소개)   
2. [기술스택](#기술스택)   
3. [빌드 및 사용법](#빌드-및-사용법)   
4. [문의](#문의)   


<br>


## 소개
- 데브코스 6기 8회차 9팀의 2차 프로젝트로 사용된 레포지토리 입니다.<br>


- 실제 자금 없이 안전하게 가상화폐 투자를 체험할 수 있는 모의투자 플랫폼입니다. 회원별 지갑이 자동 생성되며, 비트코인, 이더리움, 도지코인 등 주요 코인의 실시간 시세를 기반으로 거래할 수 있는 서비스를 제공합니다.


<br>


## 기술스택
### 백엔드
- [`Spring boot`](https://spring.io/)
- [`JWT`]
- [`OAuth2.0`]
- [`Spring Security`]
- [`Spring Data JPA`]
- [`REST API`]
- [`PostgreSQL(supabase 사용)`](https://supabase.com/)


### 프론트 엔드
- [`TypeScrpt`]
- [`Next.js`](https://nextjs.org/)
- [`Tailwind CSS`](https://tailwindcss.com/)


### 배포 
- [`Render`](https://render.com/)


<br>


## 빌드 및 사용법
### 서비스 접속
- 실제 서비스는 아래 URL에서 접속 가능합니다.<br>
  [https://peuronteuendeu.onrender.com](https://peuronteuendeu.onrender.com)


### 로컬 빌드 및 실행 
1. 깃허브 주소를 프로젝트 폴더로 다운 받는다
<br>ex) ``
git clone project-url .``


2. 빌드 실행 <br> 
### Unix / macOS
```bash
cd project_dir/backend


# 1. Clean & Build
./gradlew clean build


# 2. Run the JAR
java -jar build/libs/your-app-name-<version>.jar
