# 삼성SDS 월패드 RS485 Add-on (엘리베이터 호출 지원)

![Supports aarch64 Architecture][aarch64-shield] ![Supports amd64 Architecture][amd64-shield] ![Supports armhf Architecture][armhf-shield] ![Supports armv7 Architecture][armv7-shield] ![Supports i386 Architecture][i386-shield]

## 소개

* 삼성SDS 월패드를 사용하는 집에서, RS485를 이용해 여러 장치들을 제어할 수 있는 애드온입니다.
* 추가로 현관 스위치를 대신하여 엘리베이터를 호출하는 기능이 있습니다.

### 지원 장치

* 기본 연결: 해당 장치가 이미 월패드와 연결되어 있는 경우에 가능합니다.
  * 전등
  * 난방
  * 환기 (전열교환기)
  * 대기전력차단, 콘센트별 실시간 전력사용량
  * 가스밸브 (상태 확인)
  * 실시간 에너지 사용량 (전기, 가스, 수도)

* 현관 스위치 대체 시 추가 연결
  * **기존 현관 스위치의 RS485 연결을 분리하고, Configuration에서 entrance_mode를 "full" 로 변경하면 사용 가능합니다.**
  * 엘리베이터 호출
  * 가스밸브 잠금

## 설치

### 1. 연결

* 엘리베이터 호출을 지원하려면 RS485 패킷을 한 Byte씩 즉시 확인할 수 있어야 하므로, EW11 등의 소켓 통신 사용 시 엘리베이터 호출 동작은 성공하지 못합니다.

* 엘리베이터 호출 기능을 이용하려면, 먼저 현관에 달려있는 현관 스위치의 RS485 연결을 끊어야 합니다.
  * 일괄소등/가스밸브 차단/엘리베이터 호출 중 한 가지 기능이라도 있는 스위치가 연결되어 있으면 안됩니다.
  * 그러기 전에 "minimal" 모드를 시도해볼 수 있지만, 호출에 수십 초 이상 걸리거나 실패할 수 있습니다.

* HA가 동작하는 장치에 USB to RS485로 직접 연결하기 어려우면, 별도의 Raspberry PI에 RS485를 연결하여 이 애드온을 실행할 수도 있습니다.

### 2. 애드온 설치, 실행

#### 방법 1: HA Addon으로 설치, 실행

1. 홈어시스턴트의 Supervisor (혹은 Hass.io) --> Add-on store에서 우상단 메뉴( ⋮ ) 를 누른 후 "repositories" 선택합니다.
2. "Add repository" 영역에 "https://github.com/n-andflash/ha_addons/" 입력 후 ADD 를 누릅니다.
3. 하단에 나타난 "Samsung SDS RS485 Addon with Elevator Call" 을 선택합니다.
4. "INSTALL" 버튼을 누른 후 "START" 가 나타날 때까지 기다립니다. (수 분 이상 걸릴 수 있습니다)
  * 설치 중 오류가 발생하면 Supervisor -> System 의 System log 최하단을 확인해봐야 합니다.
5. "START" 가 보이면, 시작하기 전에 "Configuration" 페이지에서 환경에 맞게 설정을 구성 후 "SAVE" 를 누릅니다.
6. "Info" 페이지로 돌아와서 "START" 로 시작합니다.
  * 첫 시작 시 회전 애니메이션이 사라질 때까지 기다려주세요.
7. "Log" 페이지에서 정상 동작하는지 확인합니다.

#### 방법 2: 별도 장치에서 설치, 실행

* 작성중
* 일단은, options\_example.json 을 적절히 수정 후 run_standalone.sh 를 실행하면 됩니다.

### 3. MQTT 통합 구성요소 설정

* 버전 2부터 MQTT discovery를 지원하므로, yaml 파일을 일일이 설정할 필요가 없습니다.
* 통합 구성요소 페이지에 MQTT가 있고, [ ⋮ ] 를 클릭했을 때 "새로 추가된 구성요소를 활성화" 되어있다면, HA에 sds_XXX 형식의 장치들이 자동으로 추가됩니다.
* 작성중...

### 4. Lovelace 구성

* 작성중...

## 설정

* 처음 사용하시기 전에, 아래 배경색으로 강조된 다음 옵션들을 확인해 주세요.
  * serial_mode
  * entrance_mode
  * serial: port (USB to RS485 사용 시)
  * socket: address (EW11 사용 시)
  * mqtt: server
  * mqtt: need_login (Mosquitto 로그인 필요 시)

### mode:
#### `serial_mode` (serial / socket)
* serial: USB to RS485 혹은 TTL to RS485를 이용하는 경우
* socket: EW11을 이용하는 경우

#### `entrance_mode` (off / minimal / full)
* full: 현관 스위치가 없거나 연결을 끊은 경우, 이 애드온이 완전한 현관 스위치로 동작합니다.
* minimal: 현관 스위치가 있는 상황에서, 엘리베이터 호출이 필요한 경우만 강제로 끼워넣습니다. 성공률이 매우 낮아서 수십 초 이상 걸리는 경우도 있습니다. max_retry를 적절히 설정하세요.
* off: 현관 스위치 관련 기능을 비활성화 합니다. 일반적인 월패드 애드온으로만 동작합니다.

#### wallpad\_mode (on / off)
* on: 일반적인 월패드 애드온 기능
* off: 기존 애드온과 함께 쓰고 싶을 때. 이게 정상동작하는지 아직 테스트되지 않음

### serial:
* serial\_mode 가 serial 인 경우에만 필요합니다.

#### `port`
* Supervisor -> System -> HARDWARE 버튼을 눌러 serial에 적혀있는 장치 이름을 확인해서 적어주세요.
* USB to RS485를 쓰신다면 /dev/ttyUSB0, TTL to RS485를 쓰신다면 /dev/ttyAMA0 일 가능성이 높습니다.
* 단, 윈도우 환경이면 COM6 과 같은 형태의 이름을 가지고 있습니다.

#### baudrate, bytesize, parity, stopbits (기본값 9600, 8, E, 1)
* 기본값으로 두시면 됩니다.
* 사용 가능한 parity: E, O, N, M, S (Even, Odd, None, Mark, Space)

### socket:
* serial\_mode 가 socket 인 경우에만 필요합니다.

#### `address`
* EW11의 IP를 적어주세요.

#### port (기본값: 8899)
* EW11의 포트 번호를 변경하셨다면 변경한 포트 번호를 적어주세요.

### MQTT:

#### discovery (true / false)
* false로 변경하면 HA에 장치를 자동으로 등록하지 않습니다. 필요한 경우만 변경하세요.

#### prefix (기본값: sds)
* MQTT topic의 시작 단어를 변경합니다. 기본값으로 두시면 됩니다.

#### `server`
* MQTT broker (Mosquitto)의 IP를 적어주세요. 일반적으로 HA가 돌고있는 서버의 IP와 같습니다.

#### port (기본값: 1883)
* Mosquitto의 포트 번호를 변경하셨다면 변경한 포트 번호를 적어주세요. 

#### `need_login`
* Mosquitto에 login이 필요하도록 설정하셨으면 (anonymous: false) true로 수정해 주세요.

#### user, passwd
* need_login이 true인 경우 Mosquitto의 아이디와 비밀번호를 적어주세요.

### rs485:
#### max_retry (기본값: 20)
* 실행한 명령에 대한 성공 응답을 받지 못했을 때, 몇번까지 재시도할지 설정합니다. 특히 "minimal" 모드인 경우 큰 값이 필요하지만, 3회 재시도에 2초 정도 걸리므로, 너무 크지 않은 값을 설정하세요.

#### early\_response (기본값: 2)
* 현관 스위치로써 월패드에게 응답하는 타이밍을 조절합니다. 0~2. 특히 "minimal" 모드의 성공률에 약간 영향이 있습니다 (큰 기대는 하지 마세요).
  
#### dump\_time (기본값: 0)
* 0보다 큰 값을 설정하면, 애드온이 시작하기 전에 입력한 시간(초) 동안 log로 RS485 패킷 덤프를 출력합니다.
* SerialPortMon으로 RS485를 관찰하는 것과 같은 기능입니다.
* 버그/수정 제보 등 패킷 덤프가 필요할 때만 사용하세요.

### log:
#### to_file (true / false)
* false로 설정하면 로그를 파일로 남기지 않습니다.

#### filename (기본값: /share/sds_wallpad.log)
* 로그를 남길 파일 이름을 지정합니다.
* 로그는 매일 자정에 새 파일로 저장되며, 7개까지 보관됩니다.

## 지원

[HomeAssistant 네이버 카페 (질문, 수정 제안 등)](https://cafe.naver.com/koreassistant)

[Github issue 페이지 (버그 신고, 수정 제안 등)](https://github.com/n-andflash/ha_addons/issues)

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
