# SKT 고해상도 4CH 데이터 로거 / Edge Gateway 미들웨어 개발

## Overview

[TI ADS1274](http://www.ti.com/product/ADS1274)칩과 [SAMSUNG ARTIK710]()의 상위레벨 MCU 및 Pre-5G 통신 모듈을 탑재한 고성능 데이터 로거

### [TI ADS1274](http://www.ti.com/product/ADS1274)

ADS1274의 주요 특징은 다음과 같다.

#### Features

- Simultaneously Measure Four Channels
- Up to 144kSPS Data Rate

#### AC Performance

- 70kHz Bandwidth
- 111dB SNR (High-Resolution Mode) –108dB THD

#### DC Accuracy

- 0.8µV/°C Offset Drift
- 1.3ppm/°C Gain Drift

#### Selectable Operating Modes:

- High-Speed: 144kSPS, 106dB SNR
- High-Resolution: 52kSPS, 111dB SNR
- Low-Power: 52kSPS, 31mW/ch
- Low-Speed: 10kSPS, 7mW/ch
- Linear Phase Digital Filter
- SPI™ or Frame-Sync Serial Interface
- Low Sampling Aperture Error
- Modulator Output Option (digital filter bypass)
- Analog Supply: 5V
- Digital Core: 1.8V
- I/O Supply: 1.8V to 3.3V

#### APPLICATIONS

- Vibration/Modal Analysis
- Multi-Channel Data Acquisition
- Acoustics/Dynamic Strain Gauges, Pressure Sensors

### ARTIK 710

#### Key features

- High performance, 8-core, 64-bit ARM® Cortex® A-53 processor with Wi-Fi®, Bluetooth®, ZigBee®, Thread
- ARM MALI™ GPU for multimedia, graphics applications
- 1GB RAM, 4GB flash (eMMC)
- Enterprise-class security with hardware secure element and Secure OS (710s)
- Ubuntu Linux package with multimedia, connectivity, graphics, power management and security libraries

#### Use cases

- Factory automation
- Smart home gateway
- Building Automation controllers
- Multimedia applications

## Middleware Overview

- 미들웨어는 ARTIK SDK로 빌드되어 있으며, ADS1274에서 FrameSync 모드로 데이터를 수신한다.
- `ADS1274`에서 SPI로 수신한 미들웨어는 IPC Share Memory기법으로 데이터를 송신한다.
- 이후 수집된 데이터는 UDP 소켓으로 `7777`번 포트로 바인딩되어 로컬호스트 `nodeJS`의 `dataGram`으로 수신한다.
- 미들웨어 바이너리는 `mcdl-fsync-udp`로 `nodeJS`의 `child_process`로 실행되고, 시작명령(`new Buffer([115, 115, 13, 10])`)을 수신대기한다.
- 수신된 고속데이터는 `binary-parser` 모듈로 `JSON` 배열로 파싱한다.
- 파싱된 데이터는 아래의 `ads1274.js` 모듈을 통해 `voltageOut`, `gravityOut`, `jsonRMSData`, `jsonRawData`로 반환하고, `mqttHandler` 클래스 모듈을 통해 서버로 전송한다.

### `mcdl-middleware` 모듈

#### `GPIO` Pin map

```c
#define PWDN1 128
#define PWDN2 129
#define PWDN3 130
#define PWDN4 46
#define MODE0 14
#define MODE1 41
#define ADCLKDIV 25
#define FORMAT0 0
#define FORMAT1 26
#define FORMAT2 27
#define AGPIO0 61
#define ADSYNC 43
#define ADFSYNC 50
#define ADSCK 73
#define ADDOUT 75
#define ADDIN 76
#define SW403 30
#define SW404 32
```

### 애플리케이션 Cluster/Worker

#### `bin/ads1274`

- worker: `src/mcdlADC.js`

#### `bin/mcdl-ipc`

- worker: `src/mcdlIPC.js`

#### IPC event emitter

- worker: `src/nodeIPCEmitter.js`

#### IPC Processor

- worker: `src/nodeIPCProcessor.js`

#### Data Parser

- worker: `src/nodeDataParser.js`

### `ads1274.js` Node 모듈 API

#### `voltageOut(dataArray, [callback])`

- `dataArray` ADC에서 받은 bitArray의 `uint32`로 변환한 값 배열
- `callback` - `function (err, vout)`
- `vout` : 전압 값으로 환산 (ANPP : 5v), `{vout1: [double Array], vout2: [double Array], vout3: [double Array], vout4: [double Array] }`, 단위 (V)

#### `gravityOut(dataArray, [callback])`

- `dataArray` ADC에서 받은 bitArray의 `uint32`로 변환한 값 배열
- `callback` - `function (err, gout)`
- `gout` : `ADXL1002` 센서의 전압특성 offset : 1.69v, gain : 26mv/G (AIN=3.3V)을 반영, `{gout1: [double Array], gout2: [double Array], gout3: [double Array], gout4: [double Array] }`, 단위(G)

#### `jsonRMSData(dataArray, [callback])`

- `dataArray` ADC에서 받은 bitArray의 `uint32`로 변환한 값 배열
- `callback` - `function (err, JSONData)`
- `JSONData` : `{messageType: ${messageType}, deviceName: ${deviceName}, timestamp: new Date().getTime(), frequency: ${smaplingRate}, channel1_rms: rms[0], channel2_rms: rms[1], channel3_rms: rms[2], channel4_rms: rms[3]}`
  - `rms` : Math.js의 `math.std()` 함수로 RMS 계산한 결과 값 (`double`)

#### `jsonRawData(dataArray, [callback])`

- `dataArray` ADC에서 받은 bitArray의 `uint32`로 변환한 값 배열
- `callback` - `function (err, JSONData)`
- `JSONData` : `{messageType: ${messageType}, deviceName: ${deviceName}, timestamp: new Date().getTime(), frequency: ${smaplingRate}, channel1_shaft_x: gout.gout1, channel2_shaft_x: gout.gout2, channel3_shaft_x: gout.gout3, channel4_shaft_x: gout.gout4`
  - `gout` :`gravityOut(dataArray, [callback])`의 결과 값 배열, 단위(G)

### SMIC 테스트베드 설정

#### SMIC IP address

- `10.10.15.164/24` : MCDL Gateway
- `10.10.15.165/24` : APROS Gateway
- `10.10.15.122/24` : Dev PC

#### SMIC Gateway/Edge Server Address

- gw: `10.10.15.1`
- edge: `10.10.15.201`

#### `/etc/network/interfaces` 수정

```bash
auto lo eth0
iface lo inet loopback

iface eth0 inet static # LTE 라우터 네트워크
        address 192.168.50.101/24
        gateway 192.168.50.1
        dns-nameserver 8.8.8.8

iface eth0 inet static # SMIC Gateway 네트워크
        address 10.10.15.164/24
        gateway 10.10.15.1
```

### Remote Check Update

[reference](https://stackoverflow.com/questions/3258243/check-if-pull-needed-in-git)

먼저 원격 refs를 최신 상태임을 확인하기 위해 `git remote update`를 수행한다. 그러면 다음과 같은 여러 가지 작업 중 하나를 수행 할 수 있다:

- `git status -uno`는 트래킹하고있는 브랜치가 이전 커밋인지, 이후 커밋인지 또는 분기되었는지를 알려준다. 아무 옵션도 없으면 로컬과 원격이 동일하다.
- `git show-branch * master`는 이름이 'master'로 끝나는 모든 브랜치 (예 : master 및 origin / master)에서 커밋을 보여준다.

`git remote update` (`git remote -v update`)와 함께`-v`를 사용하면 어떤 브랜치가 업데이트되었는지 알 수 있으므로 더 이상 추가 명령이 필요하지 않다.

그러나 스크립트 또는 프로그램에서 이 작업을 수행하면 true/false 값으로 끝나는 것처럼 보인다. 현재 HEAD 커밋과 추적중인 지점의 헤드 사이의 관계를 확인할 수 있는 방법이 있지만 네 가지 가능한 결과가 있기 때문에 true/false 로 줄일 수는 없다. 그러나 `pull --rebase`를 수행 할 준비가 된 경우 "local is behind"및 "local has diverted"을 "need to pull"로 처리하고 다른 두 개를 "don't need to pull"으로 처리 할 수 있다.

`git rev-parse <ref>`를 사용하여 모든 refs의 커밋 ID를 얻을 수 있으므로 master 및 origin/master에 대해 이 작업을 수행하고 비교할 수 있다. 만약 이들이 서로 동일하면 브랜치도 동일하다. 이들이 동일하지 않으면, 어느 것이 다른 것보다 앞서 있는지 알아야 한다. `git merge-base master origin / master`를 사용하면 두 브랜치의 공통 조상을 알 수 있으며 분기되지 않은 경우 둘 중 하나와 동일하다. 세 가지 다른 ID를 얻으면 가지가 분기 된 것으로 판단하면 된다.

예를들어 스크립트에서 이 작업을 적절하게 수행하려면 현재 브랜치 및 트래킹중인 리모트 브랜치를 참조 할 수 있어야 한다. `/etc/bash_completion.d`의 bash 프롬프트 설정 함수에는 브렌치 이름을 얻는 데 유용한 코드가 있다. 그러나 실제로 이름을 얻을 필요는 없다. Git은 브랜치와 커밋을 참조하는 간단한 스크립트 (`git rev-parse --help`에 설명되어 있음)를 가지고 있다. 특히 현재 브렌치에는`@`를 사용하고 (HEAD 가 분리되지 않은 경우) 업스트림 브렌치에는`@{u}`를 사용할 수 있다 (예 :`origin/master`). 따라서`git merge-base @ @{u}`는 현재 브랜치와 업스트림 브렌치의 (해시)커밋을 반환하고, `git rev-parse @`및`git rev-parse @{u}`은 두 가지의 해시를 제공한다. 이 내용은 아래 스크립트에서 요약할 수 있다.

```bash
#!/bin/sh

UPSTREAM=${1:-'@{u}'}
LOCAL=$(git rev-parse @)
REMOTE=$(git rev-parse "$UPSTREAM")
BASE=$(git merge-base @ "$UPSTREAM")

if [ $LOCAL = $REMOTE ]; then
    echo "Up-to-date"
elif [ $LOCAL = $BASE ]; then
    echo "Need to pull"
elif [ $REMOTE = $BASE ]; then
    echo "Need to push"
else
    echo "Diverged"
fi
```

_참고_ : 이전 버전의 git은 `@`를 단독으로 허용하지 않으므로 대신 `@{0}`을 사용해야 한다.

`UPSTREAM=${1:-'@{u}'}`을 사용하면 현재 브렌치에 대해 구성된 것과 다른 리모트 브렌치를 검사하려는 경우 업스트림 브렌치를 명시적으로 전달할 수 있다. 일반적으로 _remotename/branchname_ 형식이고 매개 변수를 지정하지 않으면 기본값은`@{u}`이다.

스크립트는 트래킹하는 브렌치를 최신 상태로 유지하기 위해 먼저 `git fetch`또는 `git remote update`를 수행했다고 가정한다.

#### ShellJS 를 사용한 자동 업데이트

위의 스크립트에서 능동적으로 리모트 브랜치를 확인하고 의존성 모듈이 추가되는 경우 다음과 같이 구성이 가능하다.

- 시스템에 `git`이 설치되어 있는지 확인한다. 없으면 실행이 안됨
- 5초의 주기로 리모트 저장소의 업데이트된 내용을 체크한다. (비동기)
  - Remote, Local, Base의 해쉬를 변수로 불러와서 pull이 요구되면 `git pull` 명령을 수행하여 새로운 내용을 업데이트 한다. (비동기)
  - 업데이트한 파일중 `package.json`이 업데이트 되었으면 `npm install` 명령을 수행하여 의존성 모듈 설치한다.

```javascript
import shell from "shelljs";

if (!shell.which("git")) {
  shell.echo("Sorry, this script requires git");
  shell.exit(1);
}

setInterval(() => {
  shell.exec("git remote -v update", { silent: true, async: true }, () => {
    const local = new Promise((resolve, reject) =>
      shell.exec(
        "git rev-parse @",
        { silent: true, async: true },
        (code, stdout, stderr) => {
          resolve(stdout);
          reject(stderr);
        }
      )
    );
    const remote = new Promise((resolve, reject) =>
      shell.exec(
        "git rev-parse @{u}",
        { silent: true, async: true },
        (code, stdout, stderr) => {
          resolve(stdout);
          reject(stderr);
        }
      )
    );
    const base = new Promise((resolve, reject) =>
      shell.exec(
        "git merge-base @ @{u}",
        { silent: true, async: true },
        (code, stdout, stderr) => {
          resolve(stdout);
          reject(stderr);
        }
      )
    );
    local.then(localHash =>
      remote.then(remoteHash => {
        if (localHash !== remoteHash) {
          console.log("remote changed: upgrade now");
          shell.exec(
            "git pull",
            { silent: true, async: true },
            (code, stdout, stderr) => {
              console.log(stdout);
              if (stdout.includes("package.json")) {
                console.log("package.json file updated. `npm install` now");
                shell.exec(
                  "npm install",
                  { silent: true, async: true },
                  (code, stdout, stderr) => {
                    console.log(stdout);
                    console.log(stderr);
                  }
                );
              }
            }
          );
        }
      })
    );
  });
}, 5000);
```

### IPC Shared Memory 데이터 공유

Node.js로 개발할 때 가장 좋은 점 중 하나는 V8 애드온 API 덕분에 JavaScript와 네이티브 C++ 코드 사이를 원활하게 이동할 수 있다는 것이다. C++로 이동하는 기능은 처리 속도에 의해 구동되는 경우도 있지만 C++ 코드가 이미 있고 JavaScript에서 사용할 수 있기를 원하기 때문에 더 자주 발생한다.

애드온에 대한 서로 다른 사용 사례는 (1) C++ 코드에서 빠르게 수행할 수 있는 프로세싱 시간을 이용하는 방법과 (2) C++와 JavaScript간에 데이터 플로우 양이 많은 경우로 분류 할 수 있다.

#### V8 vs. C++ 메모리와 데이터 사용

Node의 네이티브 애드온을 처음 사용하는 경우 이해해야하는 첫 번째 사항 중 하나는 V8이 소유하고 있는 데이터 (C++ 애드온에서 액세스 할 수 있음)와 일반 C++ 메모리 할당 간의 차이이다.

"V8 소유"라고 말하면 JavaScript 데이터를 보유하는 스토리지 셀을 말한다. 이 스토리지 셀은 V8의 C++ API를 통해 액세스 할 수 있지만 제한된 방식으로만 액세스 할 수 있으므로 일반적인 C++ 변수는 아니다. 애드온은 V8 데이터만 사용하도록 제한 할 수 있지만 일반 C++에서는 자체 변수를 생성 할 가능성이 높다. 이들은 스택 또는 heap 변수일 수 있으며 물론 V8과 완전히 독립적이다.
JavaScript에서 기본 타입(숫자, 문자열, 부울 등)은 변경할 수 없으며 C++ 애드온은 기본 JavaScript 변수와 연관된 스토리지 셀을 변경할 수 없다. JavaScript 기본 타입의 변수는 C++로 작성된 새 스토리지 셀에 재 할당 할 수 있지만 이는 데이터를 변경하면 항상 새로운 메모리 할당이 발생한다.

JavaScript(V8 스토리지 셀)간에 이 모든 데이터를 C++로 복사하는 데 드는 시간 비용은 일반적으로 C++을 처음 실행했을 때 얻을 수있는 성능상의 이점을 없애기 때문에, 낮은 프로세싱과 높은 데이터 사용량의 상황에서 데이터 복사와 관련된 레이턴시는 무조건 애드온을 비동기식 디자인을 고려할 수 밖에 없도록하게 만든다.

#### V8 메모리와 비동기 애드온

비동기 애드온에서는 worker 스레드에서 대부분의 C++ 처리 코드를 실행한다. 비동기 애드온의 핵심 테넌트는 이벤트 루프 스레드 외부에서 V8 (JavaScript) 메모리에 액세스 할 수 없다는 것이다. 이것은 다음 문제로 이어진다. 많은 양의 데이터가있는 경우 작업자 스레드가 시작되기 전에 해당 데이터를 V8 메모리에서 이벤트 루프 스레드의 애드온 기본 주소 공간으로 복사해야한다. 마찬가지로 워커 스레드에 의해 생성되거나 수정된 모든 데이터는 콜백에서 이벤트 루프에서 실행되는 코드에 의해 V8로 다시 복사되어야힌다. 많은 데이터 처리량을 가진 Node.js 애플리케이션을 만들려면 이벤트 루프 데이터 복사에 많은 시간을 소비하지 않아야한다.

따라서 일반적인 V8 스토리지를 사용하면 두 가지 관련된 문제가 있다.

- 동기식 애드온으로 작업 할 때 데이터를 변경/생성하지 않는 한 V8 스토리지 셀과 일반 C++ 변수간에 데이터를 이동하는 데 많은 시간을 소비해야하므로 효율이 적음.

- 비동기 애드온으로 작업 할 때는 이벤트 루프에서 가능한 짧은 시간을 보내는 것이 이상적이지만 V8의 다중 스레드 제한으로 인해 이벤트 루프 스레드에서 데이터 복사를 수행해야하기 때문에 여전히 문제가 있다.

이를 해결하기 위해 `buffer`를 활용한다. `buffer`는 V8 스토리지를 사용하지 않기 때문에 C++에서 접근이 가능하다.

[reference](https://community.risingstack.com/using-buffers-node-js-c-plus-plus/)

### SHARED MEMORY FOR WRITER PROCESS

```c
#include <iostream>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
using namespace std;

int main()
{
    // ftok to generate unique key
    key_t key = ftok("shmfile",65);

    // shmget returns an identifier in shmid
    int shmid = shmget(key,1024,0666|IPC_CREAT);

    // shmat to attach to shared memory
    char *str = (char*) shmat(shmid,(void*)0,0);

    cout<<"Write Data : ";
    gets(str);

    printf("Data written in memory: %s\n",str);
    //detach from shared memory
    shmdt(str);

    return 0;
}
```

### SHARED MEMORY FOR READER PROCESS

```c
#include <iostream>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <stdio.h>
using namespace std;

int main()
{
  // ftok to generate unique key
  key_t key = ftok("shmfile",65);

  // shmget returns an identifier in shmid
  int shmid = shmget(key,1024,0666|IPC_CREAT);

  // shmat to attach to shared memory
  char *str = (char*) shmat(shmid,(void*)0,0);

  printf("Data read from memory: %s\n",str);

  //detach from shared memory
  shmdt(str);

  // destroy the shared memory
  shmctl(shmid,IPC_RMID,NULL);

  return 0;
}
```

- `ftok()` : 고유 키를 생성하는 데 사용
- `shmget()` : `int shmget (key_t, size_tsize, intshmflg);` 성공적으로 완료되면 `shmget()`은 공유 메모리 세그먼트에 대한 식별자를 반환
- `shmat()` : 공유 메모리 세그먼트를 사용하려면 `shmat()`를 사용하여 자신을 연결해야함. `void *shmat (int shmid, void *shmaddr, int shmflg);` `shmid`는 공유 메모리 ID. `shmaddr`은 사용할 특정 주소를 지정하지만 0으로 설정하면 OS가 자동으로 주소를 선택함.
- `shmdt()` : 공유 메모리 세그먼트를 완료하면 `shmdt()`를 사용하여 프로그램이 분리(detach)된다. `int shmdt (void * shmaddr);`
- `shmctl()` : 공유 메모리에서 분리해도 완전히 소멸되지 않는다. 따라서 `shmctl()`을 개체를 소멸하는 방법이 사용된다. `shmctl(int shmid, IPC_RMID, NULL);`
