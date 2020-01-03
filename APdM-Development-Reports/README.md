# A-PdM 플랫폼 단위 구성 요소

## API / Server App

### DB Model / Data Grid / Controller

- [x] Raw 데이터 모델 패킷/시간/날짜/월별/연간
- [ ] Hot(caching) 데이터 Cold 데이터 스위치
- [x] Anomaly feature 데이터 모델 개발
- [x] Diagnosis feature 데이터 모델 개발
- [x] Prognosis feature 데이터 모델 개발
- [ ] Vendor 정보 / 사용자 데이터 모델 개발
- [x] 게이트웨이 및 센서 정보 데이터 모델
- [ ] 인력 점검 IO 모델

### GraphQL API

- [x] 진동 데이터 Query API 모델 개발
- [x] 게이트웨이 및 센서정보 Query/Mutation API
- [x] Anomaly result Query/Mutation API
- [x] Diagnosis result Query/Mutation API
- [x] Prognosis result Query/Mutation APPI
- [ ] Vendor 정보 / 사용자 Query/Mutation API (context api)
- [ ] 인력 점검 IO Query/Mutation API

### Socket 통신 / MQTT 브로커

- [ ] Web push notification
- [x] Raw 데이터 실시간
- [ ] 진단 데이터 실시간

## Client 웹애플리케이션 플랫폼 (Cloud)

- [ ] 단위테스트 모듈 및 통합 테스트 모듈 구축
- [ ] CI/CD 구축
- [ ] 플랫폼 인증 절차 구축

### Dashboard Component 개발

- [ ] Test module 개발 (jest)
- [ ] 상태 모니터링 컴포넌트 개발
- [ ] 설비 장비 위치 식별을 위한 맵 컴포넌트 개발
- [ ] 설비 장비 인력 점검 일자 / 결과 조회

### Login / User Component 개발

- [ ] Test module 개발 (jest)
- [ ] 가입자 Vendor 정보별 플랫폼 스위칭
- [ ] 회원 가입 및 구독 서비스 컴포넌트 개발
- [ ] Vendor 최상위 운영자 정보 조회 / 담당자 정보 조회

### Gateway 및 MQTT Setup관련 컴포넌트 개발

- [ ] Test module 개발 (jest)
- [ ] Vendor 별 설치 정보별 Gateway 시리얼 관리
- [ ] 게이트웨이 설정 / MQTT 브로커 주소 관리
- [ ] 토큰 발행, 토큰 저장 컴포넌트
- [ ] Gateway 제어 메시징 컴포넌트

### Time-history 조회 / 특징데이터 조회 컴포넌트 개발

- [ ] Test module 개발 (jest)
- [x] Raw Data 조회 기능
- [ ] Feature Data 조회 기능

### Diagnosis feature 조회 Component 개발

- [ ] Test module 개발 (jest)
- [x] 진단 결과 시각화 Feature Data 조회 기능

### Prognosis feature 조회 Component 개발

- [ ] Test module 개발 (jest)
- [ ] 설비 장비별 RUL 표시 및 현재 위치 표시

# 분석 모듈 Core (Diagnosis / Prognosis)

## Anomaly detection module

- [x] 시간영역 특징 추출 알고리즘 (RMS, Skerness, Kurtosis, CW based RMS)
- [ ] ISO 10816 V_RMS 추출 / Severity 추출

## Diagnosis module

### SD 기반 주파수 특징 추출

- [x] Block Hankel Matrix base SD
- [x] RPM detector modules
- [x] Harmonics extractiion
- [x] Automatic Harmonics feature extraction

### EMD 기반 주파수 특징 추출

- [ ] HHT 분석 Local maxima detection
- [ ] Harmonics feature extraction

### Harmonics Feature 기반 상태 분류

- [x] Covariance weighted analysis
- [ ] Learning based Weighting function module
- [ ] SVM 기반 상태 분석

## Prognosis modules

### Pre-processing

- [ ] Feature selection based trend
- [ ] 센서 퓨전에 의한 상태 지표 표출
- [x] Ensemble averaging

### 유사성 기반 RUL 추정기 학습

- [x] 작업 영역 군집화 (Working regime clustering) : K-means 알고리즘
- [x] 작업 영역 정규화 (WR nomalization) : 통계 분석
- [ ] 추세 분석 및 상태표시 인자 구성
- [ ] 유효성 검증

### CNN 기반 RUL 추정기 모델 구성

- [ ] 특징 시각화 모델 (이미지)
- [x] Similarity based RUL 기반 이미지 학습
- [ ] 유효성 검증

# 스마트 게이트웨이 애플리케이션

## 미들웨어 기반 MQTT 애플리케이션

### 플랫폼 제어 연동
