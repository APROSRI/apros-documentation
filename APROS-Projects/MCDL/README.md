# APROS Multi-channel Data Logger Documents

## v1: Ubuntu 16.04 Linux OS에서 ADS1274와 FrameSync 통신

### 기능 & 개발 로드맵

- [x] [ADS1274](http://www.ti.com/product/ADS1274) ADC Process (Frame Sync) High-resolution Mode
- [x] [IPC Shared Memory](https://users.cs.cf.ac.uk/Dave.Marshall/C/node27.html) Process
- [x] [N-API](https://nodejs.org/api/n-api.html) Data parser process
- [x] [N-API](https://nodejs.org/api/n-api.html) Resampling process [Polyphase FIR Resampler](https://sourceforge.net/motorola/upfirdn/home/Home/)
- [x] [Redis](https://redis.io/) Cache (for acquiring timestamped data)
- [x] [CBOR](https://cbor.io/) RFC 7049 Concise Binary Object Representation (JSON data model)
- [x] Defragmentation of CBOR binary data
- [x] [MQTT.js](https://github.com/mqttjs/MQTT.js) publisher

## v1: Fedora 24 Linux OS에서 DSP와 UART 통신

### 기능 & 개발 로드맵

- [x] DSP communication UART baudrate `2500000`
- [x] [N-API](https://nodejs.org/api/n-api.html) New Data parser process
- [ ] 