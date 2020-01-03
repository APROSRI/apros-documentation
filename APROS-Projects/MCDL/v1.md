# 아프로스 4채널 데이터로거 애플리케이션 (ARTIK7)

## 기능 & 개발 로드맵

- [x] [ADS1274](http://www.ti.com/product/ADS1274) ADC Process (Frame Sync) High-resolution Mode
- [x] [IPC Shared Memory](https://users.cs.cf.ac.uk/Dave.Marshall/C/node27.html) Process
- [ ] [N-API](https://nodejs.org/api/n-api.html) Data parser process
- [x] [N-API](https://nodejs.org/api/n-api.html) Resampling process [Polyphase FIR Resampler](https://sourceforge.net/motorola/upfirdn/home/Home/)
- [x] [Redis](https://redis.io/) Cache (for acquiring timestamped data)
- [x] [CBOR](https://cbor.io/) RFC 7049 Concise Binary Object Representation (JSON data model)
- [ ] Defragmentation of CBOR binary data
- [x] [MQTT.js](https://github.com/mqttjs/MQTT.js) publisher

## 애플리케이션 API

[API 문서](API.md) 참고

## 설치 방법

### 설치스크립트

#### SSH키 생성

기존에 키가 설치되어 있으면 Overwrite 프롬프트가 있음

- `# ssh-keygen -b 2048 -t rsa -f /root/.ssh/id_rsa -q -N ""`

공개키를 [github](https://github.com/eunchurn/mcdl-gateway-middleware) 저장소의 'Setting' > 'Deploy keys'에 추가한다. 하지만 이 저장소는 Private 저장소이기 때문에 공개키 `/root/.ssh/id_rsa.pub`을 현재 저장소 소유자(eunchurn.park@aproskorea.com)으로 보내서 등록요청을 한다.

이 Deploy key는 추후 OTA등 수행에 필수

#### 시스템 설치 스크립트

- node package manager: [nvm](https://github.com/nvm-sh/nvm), [script](scripts/nvm-install.sh)
- node install v10.16.3 LTS: `nvm install 10.16.3` ARTIK7 Ubuntu OS는 Node가 설치되어 있으나, 버전이 낮기 때문에 [nvm](https://github.com/nvm-sh/nvm)으로 업그레이드를 수행
- Redis: `# apt install redis-server`
- git : `# apt install git`

### Polyphase FIR Resampler (N-API binary module)

- [Boost C++ Library](https://www.boost.org/)와 [Node.js C/C++ Addons - N-API](https://nodejs.org/api/n-api.html) 사용
- Motorola 오픈소스 프로젝트 [Polyphase FIR Resampler](https://sourceforge.net/motorola/upfirdn/home/Home/) 함수 `upfirdn`를 사용
- [SignalResampler](https://github.com/terrygta/SignalResampler)
- prerequisites: C++ Boost Library 설치 아래 참조

```bash
apt install libboost-dev libboost-all-dev
```

#### Resampling 프로세스 연산 시간 체크: Downsampling 13k to 11k

```bash
npm run resample
```

ARTIK에서 소요시간 1채널 데이터의 경우 **0.09초** 확인, 4채널의 경우 Sequencial process의 경우 **0.36초** 소요 예상

```
resampling process start:  2019-09-18T08:22:20.270Z length:  13000
resampling process   end:  2019-09-18T08:22:20.360Z length:  11000
```

#### 함수 사용법

- Javascript 엔진과 C++ 프로세스 성능 향상을 위해 [**TypedArray**](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)를 사용한다.
- **TypedArray**는 `Float32Array`를 사용한다. `const TypedData = Float32Array.from(Array)`

```javascript
import mcdl from "@build/mcdlresampler.node";

const outputData = mcdl.resampler(inputSampling, outputSampling, inputData);
```

#### 자동화 스크립트

- [ ] 자동화 설치 스크립트: [`install.sh`](scripts/install.sh)
- [x] mcdl 시스템 서비스: [`mcdl.sh`](scripts/mcdl.sh)
- [ ] OTA 시스템 서비스: [`ota.sh`](scripts/ota.sh)

#### 애플리케이션 설치

- 설치 위치: `/root/app`
- 설치 방법: `git clone git@github.com:eunchurn/mcdl-gateway-middleware.git app`
- 설치 : `cd app && npm install`
- N-API 모듈은 `build` 디렉토리에 생성된다.

## 설정

### Options

데이터로거별 설정파일은 `.env`파일에서 수정한다.

- **MQTT_HOST**: MQTT 호스트 주소
- **MQTT_QOS**: MQTT 전송 QoS
- **MQTT_USERNAME**: MQTT 사용자 이름 (not required)
- **MQTT_PASSWORD**: MQTT 사용자 비밀번호 (not required)
- **MQTT_TOPIC_BASE**: MQTT base 토픽 `v1/gw`
- **MQTT_TOPIC_EDGE**: MQTT gateway 이름 토픽 `/EdgeGW`
- **MQTT_TOPIC_TELEMETRY**: MQTT 전송 방식 토픽 `/telemetry`
- **MQTT_TOPIC_COMMAND**: MQTT 제어 명령 수신 토픽 `/command`
- **MQTT_TOPIC_RESPONSE**: MQTT 제어 명령 응답 토픽 `/commandresponse`
- **MQTT_PUB_INTERVAL**: MQTT 송신 주기 (ms)
- **MQTT_COMPRESSION**: MQTT CBOR 압축 `1`: true, `0`: false
- **DATA_RAW**: Raw 데이터 전송 `1`: true, `0`: false
- **RESAMPLING**: 가변 Sampling 주기의 데이터를 Resampling 여부 `1`: true, `0`: false
- **FIXED_FS**: 고정 샘플링 주파수 (Hz)
- **DEVICE_NAME**: 게이트웨이 디바이스 이름 `5GISMIANAMR0001`
- **TEMP_DEVICE_NAME**: 온습도 센서 게이트웨이 디바이스 이름 `5GISMIANLINE0001`
- **OTA_CHECK_INTERVAL**: OTA 업데이트 체크 주기 (ms)
