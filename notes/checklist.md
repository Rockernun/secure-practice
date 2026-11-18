# secure-practice 세팅 체크리스트

새 Spring Boot 프로젝트에 보안 세팅, 시큐어 코딩, 테스트를 빠르게 이식하기 위한 가이드라인이다.

## 0주차 · 환경 · 저장소 · 도구

### 런타임
- [ ] **JDK를 패치된 21.x LTS로 고정**
  - 적용: `sdk install java 21.0.13.0+9-tem && sdk default java 21.0.13.0+9-tem`
  - 왜: 2026.1 CPU에서 수정된 OpenJDK Base64 디코더 메모리 결함(자바포셸) 문제를 방지하기 위함이다. 공격 가능성은 낮지만, 패치 전 런타임은 외부 노출 서비스에서 즉시 공격 대상이다.

- [ ] **Gradle 툴체인으로 JDK 버전 선언**
  - 적용: `build.gradle` → `java { toolchain { languageVersion = JavaLanguageVersion.of(21) } }`
  - 왜: CI, 개발자 PC 간 JDK 불일치를 방지하기 위함이다.

### 저장소
- [ ] **민감 파일을 `.gitignore`로 사전 차단한다.**
  - 적용: `.env`, `.env.*`, `application-local.*`, `secrets/`, `*.pem`, `*.key`, `*.p12`, `*.jks` 추가한다.
  - 왜: 과거 쿠팡(2025.11), 티빙(2026.5)에서 서명키와 접속키가 소스에 남아 유출된 사례가 있어서 커밋 가능성 자체를 차단하기 위함이다.

- [ ] **Claude Code가 민감 파일을 못 읽게 deny 처리한다.**
  - 적용: `.claude/settings.json` → `permissions.deny: ["Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"]`
  - 왜: AI 리뷰 및 생성 중 비밀값이 프롬프트에 섞여 외부로 흘러나가는 공급망 공격(Mini Shai-Hulud 류)을 사전 차단하기 위함이다.

- [ ] **커밋 전 `gitleaks`로 사후 검증한다.**
  - 적용: `brew install gitleaks`; 커밋 전 `gitleaks git --no-banner`; CI에도 동일 명령을 추가한다.
  - 왜: `.gitignore`는 실수 여지가 매우 크기 때문에 사후 검증 한 겹을 더해 오류를 잡기 위함이다.

### 정적 분석
- [ ] **SpotBugs + FindSecBugs** (바이트코드 기반)
  - 적용: `build.gradle` plugins → `id 'com.github.spotbugs' version '6.0.26'`
  - 적용: dependencies → `spotbugsPlugins 'com.h3xstream.findsecbugs:findsecbugs-plugin:1.13.0'`
  - 설정: `spotbugs { effort = 'max'; reportLevel = 'medium'; ignoreFailures = true }` (초기엔 빌드 안 깨지게)
  - 왜: Spring의 리플렉션, AOP 흐름 추적에 강하다. Semgrep과 서로 사각지대를 보완하기 위함이다.

- [ ] **Semgrep** (소스 패턴 기반)
  - 적용: `brew install semgrep`; 주기 실행 `semgrep --config auto`
  - 왜: SQL 문자열 조립, 로그 인젝션 등 문자열 기반 결함에 강하기 때문이다.

### SBOM · 의존성 CVE
- [ ] **CycloneDX SBOM 생성**
  - 적용: `build.gradle` plugins → `id 'org.cyclonedx.bom' version '2.3.1'`
  - 설정: `cyclonedxBom { includeConfigs = ['runtimeClasspath']; projectType = 'application' }`
  - 왜: 과거 롯데카드(2025.8)에서 자산 목록에 빠진 서버가 7년 묵은 CVE로 뚫린 사례가 있다. 라이브러리 수준에서 같은 실수를 방지하기 위함이다.

- [ ] **Trivy로 SBOM·이미지 CVE 스캔**
  - 적용: `brew install trivy`; `./gradlew cyclonedxBom && trivy sbom build/reports/bom.json`
  - 왜: SBOM만 있고 스캔이 없으면 자산은 보이지만 위험은 모르기 때문이다.

## 1주차 · 공격 표면 (예정)

## 2주차 · 입력 검증·인젝션 (예정)

## 3주차 · 인증·토큰·키 (예정)

## 4주차 · 인가·접근통제 (예정)

## 5주차 · 비밀·암호화·설정 (예정)

## 6주차 · 공급망·의존성 (예정)

## 7주차 · 예외·로깅·탐지 (예정)

## 8주차 · 위협 모델링 (예정)
