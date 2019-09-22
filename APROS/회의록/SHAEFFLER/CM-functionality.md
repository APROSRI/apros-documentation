## CM Functionality

### Signal Type (TWF, ENV)

- TWF, ENV의 정확한 정의가 필요
- TWF : Osciloscope에 있는 1x RPM에 Window를 싱크하여 보는 기능을 의미하는 것인지? 이럴 경우 FFT, Peak-finder 등의 알고리즘 들어가야 함
- ENV : Short-time Window True/Local Peak Finder 기능을 의미하는 것인지? 이럴 경우 메모리가 충분해야하며, Peak finder 알고리즘이 들어가야 함
- Transmission? : 이 기능들은 모두 연속(continous)계측의 사안으로 4H interval에 1s duration의 간헐적 계측 시스템에서는 어떻게 데이터를 전송할지 정책이 필요

### Characteristic values (standard/special)

- 센서노드에서 계산해야 한다면, 명확한 특성값 추출 알고리즘을 제시해야 함

### Automatical adjustment of amplitude

- RMS-L 0-200Hz : 0-200Hz 대역에서의 RMS값을 의미하는지? 그렇다면 LPF cutoff frequency 200Hz 를 사용하여 RMS를 구하는 것인지? 아니면 FFT 결과에서 0-200Hz 대역만 잘라서 amplitude를 Array로 저장하는지?
- RMS-H 200-FullScale : 위의 질문과 같이, HPF cutoff frequency 200Hz를 사용하여 RMS를 구하는것인지? 아니면 FFT결과에서 200-FS의 amplitude를 Array로 저장하는지?
- Transmission? : 이 기능의 경우 마찬가지로 연속계측의 사안이므로, 어떻게 결과값을 1s/4H 정책으로 전송해야하는 것인지?

### Signal transmission on event

- MCU는 상시 켜있고, Internal ADC로 이벤트 감지가 되면 TX : 이벤트에 대한 정확한 정의가 필요
- 대기전력은 MCU 소비

### Signal transmission on demend

- on demand request에서 센서 데이터 송출 : 센서 노드의 RX가 켜져있어야 함. 거의 즉각 반응 하나 배터리 소모 약간 큼
- 게이트웨이에 on demand 정보를 가지고 있고, 센서 노드가 heart beat을 보낼 때 가끔식 정보 확인 : heart beat 기간동안 딜레이가 있음. 배터리 소모 적음
