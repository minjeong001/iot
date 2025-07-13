
# 실습 4: 아두이노-라즈베리파이 UART 통신 보고서

## 1. 서론

이 프로젝트의 목표는 아두이노와 라즈베리파이 간의 **UART 통신**을 이해하고 직접 구현하는 것이다.  
UART는 전압 변환 없이 간단히 데이터를 주고받을 수 있어 **IoT 및 임베디드 시스템에서 자주 사용**된다.  
아두이노는 센서 및 제어에, 라즈베리파이는 고속 연산과 네트워크 기능에 강점을 가지므로, 두 장치의 연동은 필수적이다.

---

## 2. 실습 과정 및 코드

### (1) 아두이노 코드 업로드

```cpp
void setup() {
  Serial.begin(9600);
  Serial.println("Arduino Ready! Waiting for data from Raspberry Pi via USB...");
}

void loop() {
  if (Serial.available()) {
    char receivedChar = Serial.read();
    Serial.print("Arduino received: ");
    Serial.println(receivedChar);
  }
}
```

- Arduino IDE > Board: Arduino Uno > Port 선택 > Upload 실행

---

### (2) 라즈베리파이 연결 및 파이썬 통신 구현

#### 2-1 라즈베리파이 환경 설정
- SD카드 굽기, SSH 활성화
- 유선랜 또는 와이파이 연결
- 아두이노를 USB-A 포트에 연결 (전압 변환 불필요)

#### 2-2 cmd or PUTTY 실행
```bash 
# (ssh 아이디@ ip주소) -> 비밀번호 임력
sudo apt-get update
sudo apt-get install python3-pip
pip3 install pyserial

#오류로 인해 가상환경 사용시

mkdir my_arduino_project
cd my_arduino_project
python3 -m venv venv
source venv/bin/activate
pip install pyserial
```


#### 2-3 포트 확인 및 설정
```bash
ls /dev/ttyACM*
```

`/boot/firmware/config.txt` 파일에서 `enable_uart=1` 확인

---

### (3) 파이썬 시리얼 통신 코드

```python
import serial
import time

SERIAL_PORT = "/dev/ttyACM0"
BAUD_RATE = 9600

try:
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print(f"포트 {SERIAL_PORT} 열림")
    time.sleep(2)

    initial_response = ser.readline().decode(errors='ignore').strip()
    if initial_response:
        print(f"초기 응답: {initial_response}")
    else:
        print("초기 응답 없음")

    message_to_send = "안녕하세요 아두이노! (From Raspberry Pi)"
    ser.write(message_to_send.encode())
    time.sleep(0.1)

    response = ser.readline().decode(errors='ignore').strip()
    if response:
        print(f"응답: {response}")
    else:
        print("응답 없음")

    for i in range(1, 6):
        test_message = f"테스트 메시지 {i}번"
        ser.write(test_message.encode())
        time.sleep(0.1)
        received = ser.readline().decode(errors='ignore').strip()
        if received:
            print(f"수신: {received}")
        else:
            print("응답 없음")
        time.sleep(1)

except serial.SerialException as e:
    print(f"시리얼 포트 오류: {e}")
    print("포트 이름 또는 권한 문제일 수 있습니다.")
    print("sudo usermod -a -G dialout $USER")
    print("sudo reboot")

except Exception as e:
    print(f"예외 발생: {e}")

finally:
    if 'ser' in locals() and ser.is_open:
        ser.close()
        print("시리얼 포트 종료")
```

---

## 3. 실습 결과 요약

- UART 통신으로 문자열을 정상적으로 주고받음
- 아두이노는 수신 메시지를 echo 형식으로 응답
- 라즈베리파이에서 시리얼 포트 연결 및 통신 성공


---

## 📌 참고 사항

- `/dev/ttyACM0` 포트는 상황에 따라 달라질 수 있음
- 실행 전 `enable_uart=1` 설정 필수
- 아두이노가 먼저 USB로 연결되어 있어야 인식됨
