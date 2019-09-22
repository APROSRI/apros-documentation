## 오류 위치
- `/root/app/init.js`
  - `const mac = macaddress.one((err, mac) => { }).toUpperCase();` : 인터넷 연결이 되는 네트워크 드라이버의 MAC 주소를 얻어오는 모듈인데, 인터넷이 없는 환경에선 `null`을 반환함. 따라서 인터넷연결이 되지 않은 환경에서는 하드코딩을 해야함
  1) `/root/app/.env` 파일 수정
   : `vi /root/app/.env` 실행 후 `GATEWAY_ID=08:15:2F:59:A1:6B`과 같이 맥주소를 적어줌
  2) `/root/app/init.js` 파일 수정 12번째 라인
  - `const mac = macaddress.one((err, mac) => { }).toUpperCase();`
  -> 
  ```
  const ethMac = macaddress.one((err, mac) => { });
  if (ethMac == null) {
  	const mac = process.env.GATEWAY_ID;
  } else {
  	const mac = ethMac.toUpperCase();
  }
  ```

- 오프라인 상태에서는 MQTT 클라이언트 모두 예외 처리해야함
- 단순 바이너리 실행 모드 추가
