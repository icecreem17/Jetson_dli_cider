# jetson_dli_cider

use_functions = [
    {
        "type": "function",
        "function": {
            "name": "measure_co2",
            "description": "Reads CO2 concentration from a CM1106 sensor connected via serial port and returns the measured value in ppm."
        }
    },
    {
        "type": "function",
        "function": {
            "name": "provide_co2_info",
            "description": "Provides additional information based on the CO2 concentration, such as health effects and recommendations."
        }
    }
]
----------------

def provide_co2_info(co2_value):
    """
    CO2 농도를 기반으로 추가 정보를 생성하는 함수.

    Args:
        co2_value (str): 측정된 CO2 농도 (ppm).

    Returns:
        str: CO2 농도와 관련된 메시지.
    """
    try:
        ppm = int(co2_value)  # CO2 농도를 정수로 변환
        if ppm >= 1000:
            return f"CO2 농도는 {ppm} ppm입니다. 이 수준에서는 두통, 피로와 같은 증상이 발생할 수 있습니다. 환기를 권장합니다."
        else:
            return f"CO2 농도는 {ppm} ppm입니다. 정상 수준이며 특별한 문제가 없습니다."
    except ValueError:
        return "CO2 농도를 해석하는 데 문제가 발생했습니다."   
        
---------------------



import serial
import time
import os
import json
import gradio as gr
from openai import OpenAI

def measure_co2():
    """
    CO2 농도를 측정하여 반환하는 함수.

    Returns:
        str: CO2 농도 (ppm).
    """
    SERIAL_PORT = "/dev/ttyUSB0"  # 실제 연결된 포트로 변경 (예: COM3)
    BAUD_RATE = 9600

    try:
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(2)  # 응답 대기 시간 증가

            # 센서 응답 읽기
            response = ser.read(9).decode('utf-8')  # 응답 문자열로 디코딩
            print(f"Raw response: {response}")  # 디버깅용 응답 출력

            if response.startswith("CO2:"):
                co2_value = response.split(":")[1].strip()
                return co2_value
            else:
                print("Error reading sensor data: Unexpected response format.")
                return "Error: Unexpected response format."

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return "Error: Serial connection issue."

    except Exception as e:
        print(f"Error during measurement: {e}")
        return f"Error: {str(e)}"






import serial
import time

def measure_co2():
    """
    CO2 농도를 측정하여 반환하는 함수.

    Returns:
        int: CO2 농도 (ppm).
    """

    SERIAL_PORT = "/dev/ttyUSB0"  # 실제 연결된 포트로 변경 (예: COM3)
    BAUD_RATE = 9600

    try:
        # 시리얼 포트 초기화
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=2) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(2)  # 응답 대기 시간 증가

            # 센서 응답 읽기
            response = ser.read(9).decode('utf-8')  # 응답 문자열로 디코딩
            print(f"Raw response: {response}")  # 디버깅용 응답 출력

            # 'CO2:734'에서 숫자 부분 추출
            if response.startswith("CO2:"):
                co2_value = int(response.split(":")[1].strip())
                return co2_value
            else:
                print("Error reading sensor data: Unexpected response format.")
                return None

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return None

    except Exception as e:
        print(f"Error during measurement: {e}")
        return None


# 예시: 함수 호출
if __name__ == "__main__":
    co2_concentration = measure_co2()
    if co2_concentration is not None:
        print(f"Measured CO2 Concentration: {co2_concentration} ppm")
    else:
        print("Measurement failed.")

use_functions = [
    {
        "type": "function",
        "function": {
            "name": "measure_co2",
            "description": "Reads CO2 concentration from a CM1106 sensor connected via serial port and returns the measured value in ppm."
        }
    }
]


import serial
import time

# 시리얼 포트 설정 (포트를 실제 연결된 포트로 변경하세요)
SERIAL_PORT = "/dev/ttyUSB0"  # 또는 COM 포트 (예: COM3)
BAUD_RATE = 9600

# 센서 데이터 읽기 함수
def read_sensor_data(serial_conn):
    try:
        serial_conn.write(b'\x11\x01\x01\xED')  # CM1106에 적합한 명령어로 수정 필요
        time.sleep(1)
        response = serial_conn.read(9)  # 센서 응답 읽기 (응답 크기를 확인하세요)
        if len(response) == 9:  # 응답이 정상일 경우
            co2 = response[2] * 256 + response[3]  # CO2 데이터 해석
            print(f"CO2: {co2} ppm")
            # CO2 농도에 따른 메시지 출력
            if co2 >= 1700:
                print("⚠️ CO2 level is very high! Ventilation is necessary!")
            elif co2 >= 1200:
                print("🔄 CO2 level is elevated. Ventilation is recommended.")
            else:
                print("Operating normal.")  # 정상 동작 메시지
        else:
            print("Error reading sensor data.")
    except Exception as e:
        print(f"Error: {e}")

# 메인 루프
def main():
    try:
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1) as ser:
            print("Serial connection established.")
            time.sleep(1)
            while True:
                read_sensor_data(ser)
                time.sleep(1)  # 1초 대기
    except serial.SerialException as e:
        print(f"Serial connection error: {e}")

if __name__ == "__main__":
    main()





    

import serial
import time

def measure_co2():
    """
    CO2 농도를 측정하여 반환하는 함수.

    Returns:
        int: CO2 농도 (ppm).
    """

    SERIAL_PORT = "/dev/ttyUSB0"  # 또는 실제 연결된 포트 (예: COM3)
    BAUD_RATE = 9600

    try:
        # 시리얼 포트 초기화
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(1)

            # 센서 응답 읽기
            response = ser.read(9)  # 응답 크기를 확인 후 조정 필요
            if len(response) == 9:
                co2 = response[2] * 256 + response[3]  # CO2 데이터 해석
                return co2
            else:
                print("Error reading sensor data: Invalid response length.")
                return None

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return None

    except Exception as e:
        print(f"Error during measurement: {e}")
        return None

# 예시: 함수 호출
if __name__ == "__main__":
    co2_concentration = measure_co2()
    if co2_concentration is not None:
        print(f"Measured CO2 Concentration: {co2_concentration} ppm")

        # CO2 농도에 따른 메시지 출력
        if co2_concentration >= 1700:
            print("\u26a0\ufe0f CO2 level is very high! Ventilation is necessary!")
        elif co2_concentration >= 1200:
            print("\ud83d\udd04 CO2 level is elevated. Ventilation is recommended.")
        else:
            print("Operating normal.")
    else:
        print("Measurement failed.")





import smbus
import Jetson.GPIO as GPIO
import time
import math

# I2C 설정
I2C_BUS = 1  # Jetson I2C 버스 번호
SENSOR_ADDRESS = 0x68  # CM1106 I2C 주소
bus = smbus.SMBus(I2C_BUS)

def read_co2():
    """
    CM1106 센서에서 CO2 데이터를 읽어오는 함수.
    Returns:
        int: CO2 농도 (ppm) 또는 None (에러 발생 시).
    """
    try:
        # CM1106 센서에서 데이터 요청
        bus.write_byte(SENSOR_ADDRESS, 0x86)
        time.sleep(0.1)
        
        # 9바이트 읽기
        data = bus.read_i2c_block_data(SENSOR_ADDRESS, 0, 9)
        
        # CO2 값 계산
        co2 = data[2] << 8 | data[3]
        return co2

    except Exception as e:
        print(f"Error reading CO2: {e}")
        return None

def measure_pm25():
    """
    PM2.5 농도를 측정하여 반환하는 함수.
    Returns:
        float: PM2.5 농도 (ug/m3) 또는 None (에러 발생 시).
    """
    pin = 8
    sample_time_ms = 30000

    # GPIO 초기화
    GPIO.setmode(GPIO.BCM)
    GPIO.setup(pin, GPIO.IN)

    low_pulse_occupancy = 0
    start_time = time.time()

    try:
        # 샘플링 시간 동안 LOW 신호 지속 시간 측정
        while (time.time() - start_time) * 1000 <= sample_time_ms:
            pulse_start = time.time()
            while GPIO.input(pin) == GPIO.LOW:
                pass
            pulse_end = time.time()

            # LOW 상태 지속 시간 계산
            pulse_duration = (pulse_end - pulse_start) * 1e6  # 마이크로초 단위로 변환
            low_pulse_occupancy += pulse_duration

        # PM2.5 농도 계산
        ratio = low_pulse_occupancy / (sample_time_ms * 10.0)
        concentration = (
            1.1 * math.pow(ratio, 3) - 3.8 * math.pow(ratio, 2) + 520 * ratio + 0.62
        )
        return round(concentration, 2)

    except Exception as e:
        print(f"Error during PM2.5 measurement: {e}")
        return None

    finally:
        GPIO.cleanup()

if __name__ == "__main__":
    while True:
        # CO2 센서 읽기
        co2_concentration = read_co2()
        if co2_concentration is not None:
            print(f"CO2: {co2_concentration} ppm")
        else:
            print("Error reading CO2 sensor data.")

        # PM2.5 센서 읽기
        pm25_concentration = measure_pm25()
        if pm25_concentration is not None:
            print(f"PM2.5: {pm25_concentration} ug/m3")
        else:
            print("Error reading PM2.5 sensor data.")

        time.sleep(1)

------------------------------------------------------
import serial
import requests

# 아두이노 시리얼 포트 설정
SERIAL_PORT = '/dev/ttyUSB0'  # 아두이노가 연결된 포트 확인 후 설정
BAUD_RATE = 9600  # 아두이노와 동일한 보드레이트 설정

# 임계값 설정
THRESHOLD_1 = 1200
THRESHOLD_2 = 1700

# 디스코드 웹훅 URL
DISCORD_WEBHOOK_URL = 'https://discord.com/api/webhooks/1316656864821379113/sL8ukrsNilzQeluyonvxZSLNxT0POnQRdQHL0TtbaQklzV1faYpwHzC43UQK8AYKc9YN'


class AlertManager:
    def __init__(self):
        self.alert_1200_sent = False
        self.alert_1700_sent = False

    def reset_alerts(self):
        """알림 상태 초기화"""
        self.alert_1200_sent = False
        self.alert_1700_sent = False


def send_discord_alert(message):
    """디스코드로 알림 메시지를 전송합니다."""
    data = {"content": message}
    response = requests.post(DISCORD_WEBHOOK_URL, json=data)
    if response.status_code == 204:
        print("[디스코드 알림] 메시지가 성공적으로 전송되었습니다.")
    else:
        print(f"[디스코드 알림] 메시지 전송 실패: {response.status_code}")


alert_manager = AlertManager()
ser = None  # 초기값 설정

try:
    # 시리얼 포트 열기
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print("아두이노 연결 완료")

    while True:
        if ser.in_waiting > 0:
            line = ser.readline().decode('utf-8').strip()  # 아두이노에서 데이터 읽기
            print(f"[아두이노 데이터] {line}")

            if line.startswith("CO2:"):
                co2_value = int(line.split(":")[1])
                print(f"[센서] 현재 CO2 농도: {co2_value} ppm")

                # 임계값 확인 및 알림 전송 (한 번만 전송)
                if co2_value > THRESHOLD_2 and not alert_manager.alert_1700_sent:
                    alert_message = f"[경고] CO2 농도가 {co2_value} ppm으로 임계값 1700 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)
                    alert_manager.alert_1700_sent = True  # 알림 상태 업데이트

                elif co2_value > THRESHOLD_1 and not alert_manager.alert_1200_sent:
                    alert_message = f"[주의] CO2 농도가 {co2_value} ppm으로 임계값 1200 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)
                    alert_manager.alert_1200_sent = True  # 알림 상태 업데이트

                # 값이 임계값 아래로 떨어지면 알림 상태 초기화
                if co2_value <= THRESHOLD_1:
                    alert_manager.reset_alerts()

except Exception as e:
    print(f"[오류] {e}")

finally:
    if ser and ser.is_open:  # ser 변수가 정의되었는지 확인
        ser.close()
        print("[센서] 시리얼 포트 닫힘")


------------------------------------------
https://discord.com/api/webhooks/1316656864821379113/sL8ukrsNilzQeluyonvxZSLNxT0POnQRdQHL0TtbaQklzV1faYpwHzC43UQK8AYKc9YN
import serial
import requests
import time

# CM1106 센서 시리얼 포트 설정
SERIAL_PORT = '/dev/ttyUSB0'  # CM1106이 연결된 포트
BAUD_RATE = 9600  # CM1106 기본 전송 속도

# 임계값 설정
THRESHOLD_1 = 1200
THRESHOLD_2 = 1700

# 디스코드 웹훅 URL
DISCORD_WEBHOOK_URL = 'https://discord.com/api/webhooks/your_webhook_url_here'

def send_discord_alert(message):
    """디스코드로 알림 메시지를 전송합니다."""
    data = {"content": message}
    response = requests.post(DISCORD_WEBHOOK_URL, json=data)
    if response.status_code == 204:
        print("[디스코드 알림] 메시지가 성공적으로 전송되었습니다.")
    else:
        print(f"[디스코드 알림] 메시지 전송 실패: {response.status_code}")

try:
    # 시리얼 포트 열기
    ser = serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=1)
    print("CM1106 센서 연결 완료")

    while True:
        # 센서 데이터 읽기
        if ser.in_waiting > 0:
            data = ser.read(9)  # CM1106 데이터는 9바이트
            if len(data) == 9 and data[0] == 0x16 and data[1] == 0x04:
                co2_value = data[2] * 256 + data[3]  # CO2 농도 계산
                print(f"[센서] 현재 CO2 농도: {co2_value} ppm")

                # 임계값 확인 및 알림 전송
                if co2_value > THRESHOLD_2:
                    alert_message = f"[경고] CO2 농도가 {co2_value} ppm으로 임계값 1700 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)
                elif co2_value > THRESHOLD_1:
                    alert_message = f"[주의] CO2 농도가 {co2_value} ppm으로 임계값 1200 ppm을 초과했습니다!"
                    print(alert_message)
                    send_discord_alert(alert_message)

        time.sleep(1)  # 1초 간격으로 데이터 읽기

except Exception as e:
    print(f"[오류] {e}")

finally:
    if ser.is_open:
        ser.close()
        print("[센서] 시리얼 포트 닫힘")




------------------------------
아두이노에서 그냥 co2데이터만 읽는 코드

#include <cm1106_i2c.h>

#include <cm1106_i2c.h>

CM1106_I2C cm1106_i2c;

void setup() {
  cm1106_i2c.begin();
  Serial.begin(9600);
  delay(1000);
  cm1106_i2c.read_serial_number();
  delay(1000);
  cm1106_i2c.check_sw_version();
  delay(1000);
}

void loop() {
  uint8_t ret = cm1106_i2c.measure_result();

  if (ret == 0) {
    Serial.print("Co2 : ");
    Serial.println(cm1106_i2c.co2);
    Serial.println("Operating normal");
  }
  delay(1000);
}





























#include <SoftwareSerial.h>
 
SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11,0x01,0x01,0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;
 
void Send_CMD(void) {
  unsigned int i;
  for(i=0; i<4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}
unsigned char Checksum_cal(void) {
  unsigned char count, SUM=0;
  for(count=0; count<7; count++) {
     SUM += Receive_Buff[count];
  }
  return 256-SUM;
}
 
void setup() {
  pinMode(13,INPUT);
  pinMode(11,OUTPUT);
  Serial.begin(9600);
  while (!Serial) ;
  mySerial.begin(9600);
  while (!mySerial);
}
 
void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while(1) {
    if(mySerial.available()) { 
       Receive_Buff[recv_cnt++] = mySerial.read();
      if(recv_cnt ==8){recv_cnt = 0; break;}
    }
  } 
  
  if(Checksum_cal() == Receive_Buff[7]) {
     PPM_Value = Receive_Buff[3]<<8 | Receive_Buff[4];
     Serial.write("   PPM : ");
     Serial.println(PPM_Value);
  }
   else {
    Serial.write("CHECKSUM Error");
  }
  delay(1000);
} 
















#include <SoftwareSerial.h>

SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11, 0x01, 0x01, 0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;

void Send_CMD(void) {
  unsigned int i;
  for (i = 0; i < 4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}

unsigned char Checksum_cal(void) {
  unsigned char count, SUM = 0;
  for (count = 0; count < 7; count++) {
    SUM += Receive_Buff[count];
  }
  return 256 - SUM;
}

void setup() {
  pinMode(13, INPUT);
  pinMode(11, OUTPUT);
  Serial.begin(9600);

  while (!Serial);
  mySerial.begin(9600);
  while (!mySerial);
}

void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while (1) {
    if (mySerial.available()) {
      Receive_Buff[recv_cnt++] = mySerial.read();
      if (recv_cnt == 8) {
        recv_cnt = 0;
        break;
      }
    }
  }

  if (Checksum_cal() == Receive_Buff[7]) {
    PPM_Value = Receive_Buff[3] << 8 | Receive_Buff[4];
    Serial.write(" PPM : ");
    Serial.println(PPM_Value);
  } else {
    Serial.write("CHECKSUM Error");
  }

  delay(1000);
}




import serial
import time

# 젯슨 나노의 UART 포트에 맞게 수정 (보통 '/dev/ttyTHS1' 사용)
arduino = serial.Serial('/dev/ttyTHS1', baudrate=9600, timeout=1)

time.sleep(2)  # 아두이노 초기화 대기

while True:
    # 명령어 전송
    arduino.write(b'\x11\x01\x01\xED')
    time.sleep(1)

    # 데이터 수신
    data = arduino.read(8)
    if len(data) == 8:
        checksum = 256 - sum(data[:7]) % 256
        if checksum == data[7]:
            ppm_value = data[3] << 8 | data[4]
            print(f"PPM: {ppm_value}")
        else:
            print("Checksum Error")
    else:
        print("Data not received")





#include <SoftwareSerial.h>

SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11, 0x01, 0x01, 0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;

void Send_CMD(void) {
  unsigned int i;
  for (i = 0; i < 4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}

unsigned char Checksum_cal(void) {
  unsigned char count, SUM = 0;
  for (count = 0; count < 7; count++) {
    SUM += Receive_Buff[count];
  }
  return 256 - SUM;
}

void setup() {
  pinMode(13, INPUT);
  pinMode(11, OUTPUT);
  Serial.begin(9600);

  while (!Serial);
  mySerial.begin(9600);
  while (!mySerial);
}

void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while (1) {
    if (mySerial.available()) {
      Receive_Buff[recv_cnt++] = mySerial.read();
      if (recv_cnt == 8) {
        recv_cnt = 0;
        break;
      }
    }
  }

  if (Checksum_cal() == Receive_Buff[7]) {
    PPM_Value = Receive_Buff[3] << 8 | Receive_Buff[4];
    Serial.write(" PPM : ");
    Serial.println(PPM_Value);
  } else {
    Serial.write("CHECKSUM Error");
  }

  delay(1000);
}













unsigned char Send_data[4] = {0x11, 0x01, 0x01, 0xED}; // 데이터 요청 명령
unsigned char Receive_Buff[8];  // 센서에서 수신한 데이터를 저장
unsigned char recv_cnt = 0;
unsigned int PPM_Value;

void Send_CMD() {
  for (int i = 0; i < 4; i++) {
    Serial.write(Send_data[i]);  // 명령 전송
    delay(1);
  }
}

unsigned char Checksum_cal() {
  unsigned char SUM = 0;
  for (int count = 0; count < 7; count++) {
    SUM += Receive_Buff[count];
  }
  return 256 - SUM;
}

void setup() {
  Serial.begin(9600);  // 센서와 통신 시작 (UART 통신)
}

void loop() {
  Serial.print("Sending...");
  Send_CMD();  // CO2 데이터를 요청
  delay(10);   // 센서 응답 대기

  // 센서 데이터 수신
  recv_cnt = 0;
  while (Serial.available()) {
    Receive_Buff[recv_cnt++] = Serial.read();  // 데이터 수신
    if (recv_cnt == 8) break;  // 8바이트 수신 완료 시 종료
  }

  // 수신 데이터 처리
  if (Checksum_cal() == Receive_Buff[7]) {  // 체크섬 확인
    PPM_Value = (Receive_Buff[3] << 8) | Receive_Buff[4];  // PPM 값 계산
    Serial.print("PPM: ");
    Serial.println(PPM_Value);
  } else {
    Serial.println("CHECKSUM Error");
  }

  delay(1000);  // 1초 간격으로 데이터 요청
}


























#include <SoftwareSerial.h>

SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11, 0x01, 0x01, 0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;

void Send_CMD(void) {
  for (int i = 0; i < 4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}

unsigned char Checksum_cal(void) {
  unsigned char SUM = 0;
  for (int count = 0; count < 7; count++) {
    SUM += Receive_Buff[count];
  }
  return 256 - SUM;
}

void setup() {
  pinMode(13, INPUT);
  pinMode(11, OUTPUT);
  Serial.begin(9600);
  mySerial.begin(9600);
}

void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while (mySerial.available()) {
    Receive_Buff[recv_cnt++] = mySerial.read();
    if (recv_cnt == 8) {
      recv_cnt = 0;
      break;
    }
  }

  if (Checksum_cal() == Receive_Buff[7]) {
    PPM_Value = Receive_Buff[3] << 8 | Receive_Buff[4];
    Serial.print("PPM: ");
    Serial.println(PPM_Value);
  } else {
    Serial.println("CHECKSUM Error");
  }

  delay(1000);
}





#include <SoftwareSerial.h>

SoftwareSerial mySerial(13, 11);
unsigned char Send_data[4] = {0x11, 0x01, 0x01, 0xED};
unsigned char Receive_Buff[8];
unsigned char recv_cnt = 0;
unsigned int PPM_Value;

void Send_CMD(void) {
  unsigned int i;
  for (i = 0; i < 4; i++) {
    mySerial.write(Send_data[i]);
    delay(1);
  }
}

unsigned char Checksum_cal(void) {
  unsigned char count, SUM = 0;
  for (count = 0; count < 7; count++) {
    SUM += Receive_Buff[count];
  }
  return 256 - SUM;
}

void setup() {
  pinMode(13, INPUT);
  pinMode(11, OUTPUT);
  Serial.begin(9600);

  while (!Serial);
  mySerial.begin(9600);
  while (!mySerial);
}

void loop() {
  Serial.print("Sending...");
  Send_CMD();
  while (1) {
    if (mySerial.available()) {
      Receive_Buff[recv_cnt++] = mySerial.read();
      if (recv_cnt == 8) {
        recv_cnt = 0;
        break;
      }
    }
  }

  if (Checksum_cal() == Receive_Buff[7]) {
    PPM_Value = Receive_Buff[3] << 8 | Receive_Buff[4];
    Serial.write(" PPM : ");
    Serial.println(PPM_Value);
  } else {
    Serial.write("CHECKSUM Error");
  }

  delay(1000);
}









void setup() {
  Serial.begin(9600);  // Jetson Nano와의 시리얼 통신 시작
}

void loop() {
  int co2 = analogRead(A0);  // CO2 데이터 읽기 (예: 아날로그 핀 A0 사용)

  // CO2 데이터 전송
  Serial.println(co2);  // CO2 데이터 전송

  delay(2000);  // 2초 대기 후 다음 데이터 읽기
}






#include "DHT.h"

// DHT22 핀 및 타입 설정
#define DHTPIN 2  // DHT22 센서가 연결된 핀
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);  // Jetson Nano와의 시리얼 통신 시작
  dht.begin();
}

void loop() {
  float temperature = dht.readTemperature();  // 온도 읽기
  float humidity = dht.readHumidity();        // 습도 읽기
  int co2 = analogRead(A0);                   // CO2 데이터 읽기 (예: 아날로그 핀 A0 사용)

  // 데이터 유효성 검사
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Error reading from DHT sensor!");
    delay(2000);
    return;
  }

  // 쉼표로 구분된 데이터 전송
  Serial.print(temperature);  // 온도 데이터 전송
  Serial.print(",");
  Serial.print(humidity);     // 습도 데이터 전송
  Serial.print(",");
  Serial.println(co2);        // CO2 데이터 전송

  delay(2000);  // 2초 대기 후 다음 데이터 읽기
}







#include <SoftwareSerial.h>

SoftwareSerial mySerial(10, 11); // RX, TX 핀 설정

void setup() {
  Serial.begin(9600); // 기본 시리얼 모니터
  mySerial.begin(9600); // CM1106과 통신용 시리얼
  
  Serial.println("CM1106 CO2 Sensor Test");
}

void loop() {
  byte request[] = {0x11, 0x01, 0x01, 0xED}; // CM1106 데이터 요청 명령
  byte response[8]; // 응답 저장용 배열

  mySerial.write(request, sizeof(request)); // 센서에 데이터 요청
  delay(100); // 센서 응답 대기
  
  if (mySerial.available() >= 8) { // 응답 데이터가 충분히 도착했는지 확인
    for (int i = 0; i < 8; i++) {
      response[i] = mySerial.read(); // 데이터 읽기
    }

    // 응답 데이터에서 CO2 농도 추출 (데이터 시트 참고)
    if (response[0] == 0x16 && response[1] == 0x02) {
      int co2 = (response[2] << 8) | response[3];
      Serial.println("CO2: " + String(co2) + " ppm");
    }
  }
  delay(2000); // 2초마다 데이터 요청
}
















------------------------------------------------


use_functions = [
    {
        "type": "function",
        "function": {
            "name": "measure_pm25",
            "description": "Measures PM2.5 concentration using a GPIO pin and returns the calculated value in micrograms per cubic meter (ug/m3).",
        }
    }
]

#include "DHT.h"

// DHT22 핀 및 타입 설정
#define DHTPIN 2  // DHT22 센서가 연결된 핀
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);  // Jetson Nano와의 시리얼 통신 시작
  dht.begin();
}

void loop() {
  float temperature = dht.readTemperature();  // 온도 읽기
  float humidity = dht.readHumidity();        // 습도 읽기
  int co2 = analogRead(A0);                   // CO2 데이터 읽기 (예: 아날로그 핀 A0 사용)

  // 데이터 유효성 검사
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Error reading from DHT sensor!");
    delay(2000);
    return;
  }

  // 쉼표로 구분된 데이터 전송
  Serial.print(temperature);  // 온도 데이터 전송
  Serial.print(",");
  Serial.print(humidity);     // 습도 데이터 전송
  Serial.print(",");
  Serial.println(co2);        // CO2 데이터 전송

  delay(2000);  // 2초 대기 후 다음 데이터 읽기
}
>>파이썬 저장 코드
import serial
import pandas as pd
import time

# 시리얼 포트 설정 (Jetson Nano의 UART 포트 사용)
try:
    sensor_port = serial.Serial('/dev/ttyUSB0', 9600, timeout=2)  # Arduino 연결 포트
except serial.SerialException as e:
    print(f"Error: Could not open port. {e}")
    exit(1)

data_list = []

# 데이터 수집 및 저장 함수
def collect_and_save_data(duration=1):  # 기본값: 1분 동안 실행
    start_time = time.time()
    formatted_start_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(start_time))

    try:
        while time.time() - start_time < duration * 60:  # 정해진 시간 동안 실행
            if sensor_port.in_waiting > 0:
                raw_data = sensor_port.readline().decode('utf-8').strip()
                print(f"Raw data received: {raw_data}")
                try:
                    # 데이터가 쉼표로 구분된 3개의 값인지 확인
                    if len(raw_data.split(",")) == 3:
                        temperature, humidity, co2 = raw_data.split(",")
                        timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
                        data_list.append({
                            "Timestamp": timestamp,
                            "Temperature": float(temperature),
                            "Humidity": float(humidity),
                            "CO2_Level": float(co2)
                        })
                        print(f"Parsed: Temperature={temperature}, Humidity={humidity}, CO2={co2}")
                    else:
                        print(f"Invalid data format: {raw_data}")
                except ValueError as e:
                    print(f"Parsing error: {raw_data}. Error: {e}")
            else:
                print("No data received from sensor. Waiting...")

        # 시작 시간과 종료 시간 추가
        formatted_end_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
        data_list.append({"Timestamp": "Start Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_start_time})
        data_list.append({"Timestamp": "End Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_end_time})

        # 데이터프레임 생성 및 엑셀 저장
        df = pd.DataFrame(data_list)
        print(df.head())  # 데이터프레임 출력 (확인용)
        file_path = "/home/dli/sensor_data.xlsx"  # 저장 경로
        df.to_excel(file_path, index=False)
        print(f"Excel file saved as '{file_path}'")

    except serial.SerialException as e:
        print(f"Serial exception: {e}")
    except KeyboardInterrupt:
        print("Data collection stopped by user.")

    finally:
        if sensor_port.is_open:
            sensor_port.close()

# 프로그램 실행
if __name__ == "__main__":
    collect_and_save_data(duration=1)  # 1분 동안 데이터 수집


>> 저장된 엑셀파일 메일로 보내는 파이썬 함수
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders
import os

def send_email(file_path, sender_email, receiver_email, password):
    """
    엑셀 파일을 이메일로 전송하는 함수.
    
    Parameters:
        file_path (str): 첨부할 엑셀 파일 경로
        sender_email (str): 발신자의 Gmail 주소
        receiver_email (str): 수신자의 이메일 주소
        password (str): Gmail 앱 비밀번호
    """
    # 파일 존재 여부 확인
    if not os.path.exists(file_path):
        print(f"Error: File '{file_path}' does not exist.")
        return

    try:
        # 이메일 메시지 설정
        msg = MIMEMultipart()
        msg['From'] = sender_email
        msg['To'] = receiver_email
        msg['Subject'] = "Sensor Data (Excel File Attached)"

        # 첨부 파일 설정
        attachment = MIMEBase('application', 'octet-stream')
        with open(file_path, 'rb') as file:
            attachment.set_payload(file.read())
        encoders.encode_base64(attachment)
        attachment.add_header('Content-Disposition', f'attachment; filename={os.path.basename(file_path)}')
        msg.attach(attachment)

        # Gmail SMTP 서버에 연결
        with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
            server.login(sender_email, password)
            server.sendmail(sender_email, receiver_email, msg.as_string())

        print(f"Email sent successfully to {receiver_email} with file '{file_path}' attached.")

    except Exception as e:
        print(f"Failed to send email: {e}")

# 실행 예제
if __name__ == "__main__":
    # 엑셀 파일 경로와 이메일 설정
    file_path = "/home/your_username/sensor_data.xlsx"  # 저장된 엑셀 파일 경로
    sender_email = "your_email@gmail.com"              # 발신자의 Gmail 주소
    receiver_email = "recipient_email@gmail.com"       # 수신자의 이메일 주소
    password = "your_app_password"                     # Gmail 앱 비밀번호

    # 이메일 전송 함수 호출
    send_email(file_path, sender_email, receiver_email, password)




잭슨 나노 일대기

dli@dli:~$ python3 email.py
Traceback (most recent call last):
  File "email.py", line 1, in <module>
    import smtplib
  File "/usr/lib/python3.6/smtplib.py", line 47, in <module>
    import email.utils
  File "/home/dli/email.py", line 2, in <module>
    from email.mime.multipart import MIMEMultipart
ModuleNotFoundError: No module named 'email.mime'; 'email' is not a package
Error in sys.excepthook:
Traceback (most recent call last):
  File "/usr/lib/python3/dist-packages/apport_python_hook.py", line 72, in apport_excepthook
    from apport.fileutils import likely_packaged, get_recent_crashes
  File "/usr/lib/python3/dist-packages/apport/__init__.py", line 5, in <module>
    from apport.report import Report
  File "/usr/lib/python3/dist-packages/apport/report.py", line 21, in <module>
    from urllib.request import urlopen
  File "/usr/lib/python3.6/urllib/request.py", line 86, in <module>
    import email
  File "/home/dli/email.py", line 1, in <module>
    import smtplib
  File "/usr/lib/python3.6/smtplib.py", line 47, in <module>
    import email.utils
ModuleNotFoundError: No module named 'email.utils'; 'email' is not a package

Original exception was:
Traceback (most recent call last):
  File "email.py", line 1, in <module>
    import smtplib
  File "/usr/lib/python3.6/smtplib.py", line 47, in <module>
    import email.utils
  File "/home/dli/email.py", line 2, in <module>
    from email.mime.multipart import MIMEMultipart
ModuleNotFoundError: No module named 'email.mime'; 'email' is not a package

===============
import serial
import pandas as pd
import time
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# 시리얼 포트 설정 (Jetson Nano의 UART 포트 사용)
sensor_port = serial.Serial('/dev/ttyUSB0', 9600)  # 단일 포트로 설정

data_list = []

# 이메일 전송 함수
def send_email(file_path):
    sender_email = "your_email@gmail.com"
    receiver_email = "recipient_email@gmail.com"
    password = "your_app_password"

    msg = MIMEMultipart()
    msg['From'] = sender_email
    msg['To'] = receiver_email
    msg['Subject'] = "Sensor Data (Temperature, Humidity, CO2)"

    attachment = MIMEBase('application', 'octet-stream')
    with open(file_path, "rb") as file:
        attachment.set_payload(file.read())
    encoders.encode_base64(attachment)
    attachment.add_header('Content-Disposition', f'attachment; filename={file_path}')
    msg.attach(attachment)

    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, msg.as_string())

    print(f"Email sent to {receiver_email} with {file_path} attached.")

# 데이터 수집 및 저장
def collect_and_save_data(duration=30):
    start_time = time.time()
    formatted_start_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(start_time))

    try:
        while time.time() - start_time < duration * 60:  # 정해진 시간(30분) 동안 실행
            if sensor_port.in_waiting > 0:
                raw_data = sensor_port.readline().decode('utf-8').strip()
                print(f"Raw data received: {raw_data}")
                try:
                    if len(raw_data.split(",")) == 3:
                        temperature, humidity, co2 = raw_data.split(",")
                        timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
                        data_list.append({
                            "Timestamp": timestamp,
                            "Temperature": float(temperature),
                            "Humidity": float(humidity),
                            "CO2_Level": float(co2)
                        })
                        print(f"Parsed: Temperature={temperature}, Humidity={humidity}, CO2={co2}")
                    else:
                        print(f"Invalid data format: {raw_data}")
                except ValueError:
                    print(f"Parsing error: {raw_data}")

        # 시작 시간과 종료 시간 추가
        formatted_end_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
        data_list.append({"Timestamp": "Start Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_start_time})
        data_list.append({"Timestamp": "End Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_end_time})

        # 데이터프레임 생성 및 엑셀 저장
        df = pd.DataFrame(data_list)
        print(df.head())  # 데이터프레임 출력 (확인용)
        file_path = "sensor_data.xlsx"
        df.to_excel(file_path, index=False)
        print(f"Excel file saved as '{file_path}'")

        return file_path

    except KeyboardInterrupt:
        print("Data collection stopped by user.")

    finally:
        sensor_port.close()

# 프로그램 실행
if __name__ == "__main__":
    excel_file = collect_and_save_data(duration=1)  # 30분 동안 데이터 수집
    send_email(excel_file)  # 수집된 데이터를 이메일로 전송

    Arduino: 1.8.19 (Linux), Board: "Arduino Uno"

Sketch uses 5452 bytes (16%) of program storage space. Maximum is 32256 bytes.
Global variables use 561 bytes (27%) of dynamic memory, leaving 1487 bytes for local variables. Maximum is 2048 bytes.
An error occurred while uploading the sketch

avrdude: stk500_getparm(): (a) protocol error, expect=0x14, resp=0x00

avrdude: stk500_getparm(): (a) protocol error, expect=0x14, resp=0x00
avrdude: stk500_initialize(): (a) protocol error, expect=0x14, resp=0x00
avrdude: initialization failed, rc=-1
         Double check connections and try again, or use -F to override
         this check.

avrdude: stk500_disable(): protocol error, expect=0x14, resp=0x00


This report would have more information with
"Show verbose output during compilation"
option enabled in File -> Preferences.


#include "DHT.h"

#define DHTPIN 2      // DHT22 센서 핀
#define DHTTYPE DHT22

#define CM1106PIN 3   // CM1106 센서의 데이터 핀 (소프트웨어 시리얼 사용)

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);  // Jetson Nano와의 시리얼 통신
  dht.begin();
}

void loop() {
  // DHT22 센서 읽기
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  // CM1106 센서 읽기 (예: 아날로그 값으로 측정)
  int co2 = analogRead(A0);  // CM1106 데이터 핀 연결

  // 데이터 유효성 검사
  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Failed to read from DHT sensor!");
    delay(2000);
    return;
  }

  // 쉼표로 구분된 데이터 전송
  Serial.print(temperature);  // 온도
  Serial.print(",");
  Serial.print(humidity);     // 습도
  Serial.print(",");
  Serial.println(co2);        // CO2

  delay(2000);  // 2초 대기
}
ㅇㅇㅇㅇㅇㅇㅇ

import serial
import pandas as pd
import time

# 시리얼 포트 설정 (Jetson Nano의 UART 포트 사용)
sensor_port = serial.Serial('/dev/ttyUSB0', 9600)  # Arduino 연결 포트
data_list = []

# 데이터 수집 및 저장
def collect_and_save_data(duration=30):
    start_time = time.time()
    formatted_start_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(start_time))

    try:
        while time.time() - start_time < duration * 60:  # 정해진 시간(30분) 동안 실행
            if sensor_port.in_waiting > 0:
                raw_data = sensor_port.readline().decode('utf-8').strip()
                print(f"Raw data received: {raw_data}")
                try:
                    if len(raw_data.split(",")) == 3:
                        temperature, humidity, co2 = raw_data.split(",")
                        timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
                        data_list.append({
                            "Timestamp": timestamp,
                            "Temperature": float(temperature),
                            "Humidity": float(humidity),
                            "CO2_Level": float(co2)
                        })
                        print(f"Parsed: Temperature={temperature}, Humidity={humidity}, CO2={co2}")
                    else:
                        print(f"Invalid data format: {raw_data}")
                except ValueError:
                    print(f"Parsing error: {raw_data}")

        # 시작 시간과 종료 시간 추가
        formatted_end_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
        data_list.append({"Timestamp": "Start Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_start_time})
        data_list.append({"Timestamp": "End Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_end_time})

        # 데이터프레임 생성 및 엑셀 저장
        df = pd.DataFrame(data_list)
        print(df.head())  # 데이터프레임 출력 (확인용)
        file_path = "/home/your_username/sensor_data.xlsx"  # 저장 경로
        df.to_excel(file_path, index=False)
        print(f"Excel file saved as '{file_path}'")

    except KeyboardInterrupt:
        print("Data collection stopped by user.")

    finally:
        sensor_port.close()

# 프로그램 실행
if __name__ == "__main__":
    collect_and_save_data(duration=30)  # 30분 동안 데이터 수집



    
day 1 : 
잭슨 나노 한글 설치까지
```
import serial
import pandas as pd
import time

# 시리얼 포트 설정 (Jetson Nano의 UART 포트 사용)
sensor_port = serial.Serial('/dev/ttyUSB0', 9600)  # Arduino 연결 포트
data_list = []

# 데이터 수집 및 저장 함수
def collect_and_save_data(duration=30):
    start_time = time.time()
    formatted_start_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(start_time))

    try:
        while time.time() - start_time < duration * 60:  # 30분 동안 실행
            if sensor_port.in_waiting > 0:
                raw_data = sensor_port.readline().decode('utf-8').strip()
                print(f"Raw data received: {raw_data}")
                try:
                    # 데이터가 쉼표로 구분된 3개의 값인지 확인
                    if len(raw_data.split(",")) == 3:
                        temperature, humidity, co2 = raw_data.split(",")
                        timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
                        data_list.append({
                            "Timestamp": timestamp,
                            "Temperature": float(temperature),
                            "Humidity": float(humidity),
                            "CO2_Level": float(co2)
                        })
                        print(f"Parsed: Temperature={temperature}, Humidity={humidity}, CO2={co2}")
                    else:
                        print(f"Invalid data format: {raw_data}")
                except ValueError:
                    print(f"Parsing error: {raw_data}")

        # 시작 시간과 종료 시간 추가
        formatted_end_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
        data_list.append({"Timestamp": "Start Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_start_time})
        data_list.append({"Timestamp": "End Time", "Temperature": "-", "Humidity": "-", "CO2_Level": formatted_end_time})

        # 데이터프레임 생성 및 엑셀 저장
        df = pd.DataFrame(data_list)
        print(df.head())  # 데이터프레임 출력 (확인용)
        file_path = "/home/your_username/sensor_data.xlsx"  # 저장 경로
        df.to_excel(file_path, index=False)
        print(f"Excel file saved as '{file_path}'")

    except KeyboardInterrupt:
        print("Data collection stopped by user.")

    finally:
        sensor_port.close()

# 프로그램 실행
if __name__ == "__main__":
    collect_and_save_data(duration=30)  # 30분 동안 데이터 수집


![KakaoTalk_20241114_184028807](https://github.com/user-attachments/assets/e7bd0456-608a-486b-8ed4-6df9e4007686)
![KakaoTalk_20241114_184028807_01](https://github.com/user-attachments/assets/6f5923d1-277b-4f20-a6fa-071122d3d931)
![KakaoTalk_20241114_184028807_02](https://github.com/user-attachments/assets/53871aeb-74bb-432b-b43c-2ce575c0791e)
![KakaoTalk_20241114_184028807_03](https://github.com/user-attachments/assets/1e10cd75-019e-4310-aeea-fe65a15e5839)
![KakaoTalk_20241114_184028807_04](https://github.com/user-attachments/assets/548d8c5e-c6bb-44b4-badc-056601b73fe8)
![Screenshot from 2024-11-14 20-51-02](https://github.com/user-attachments/assets/46ddcb10-888a-4458-9901-f863b119ff4d)

```
day 2 :
1. 잭슨 나노 영상 캡쳐 이미지
```
![image](https://github.com/user-attachments/assets/4f6050d6-6fe3-41dd-886a-4f8b2a7a6888)
![Screenshot from 2024-11-14 20-59-17](https://github.com/user-attachments/assets/1ddb0609-342e-42b7-8a71-1262b339d05b)

```
2. 잭슨 나노 영상 캡쳐 동영상
```
https://github.com/user-attachments/assets/753b3ac5-ee2d-4176-bb74-15c2a446380b

```
3. 엔비디아 사진 다운로드
```
![Screenshot from 2024-11-14 21-45-16](https://github.com/user-attachments/assets/950ff074-5eb5-486b-b481-387118ec2d9f)
![Screenshot from 2024-11-14 21-45-32](https://github.com/user-attachments/assets/60e8b810-42c9-4dd8-8294-44478b930d15)
```
day 3 :
1. 엔비디아 사진 다운로드 완료 & 주피터 노트북 url생성
```
![Screenshot from 2024-11-21 18-54-55](https://github.com/user-attachments/assets/9586b14b-d830-40a7-b480-eaff1186b8e7)

```
2. 스왑 18기가 생성
```
![스왑 사진  jpg](https://github.com/user-attachments/assets/a7e28215-6e86-488c-8f4b-01feaf3d4039)
 -> reboot 중 GUI 모드로 설정하는 과정

![Screenshot from 2024-11-21 20-20-05](https://github.com/user-attachments/assets/9793be0a-ea05-4966-9041-73e2d057b8b4)

 -> 메모리가 1.9기가 에서 18기가로 늘어난 것을 볼 수 있음

```
3. 주피터 노트북 접속
```
![Screenshot from 2024-11-21 19-33-56](https://github.com/user-attachments/assets/cc3f49f8-a8ee-41e4-9002-1b09daa73b44)

```
4. 엄지 방향 사진 학습
```
![Screenshot from 2024-11-21 20-15-42](https://github.com/user-attachments/assets/a9084f30-7b82-4cbb-94a6-bd2273331c10)

```
5. 엄지 방향 사진 예측 완료
```
![Screenshot from 2024-11-21 20-23-43](https://github.com/user-attachments/assets/e84f3a5c-9297-408d-9863-99cf1b86ab61)
![Screenshot from 2024-11-21 20-24-24](https://github.com/user-attachments/assets/39db115d-eaa8-46e2-bc2b-9e46daa147e7)

#include <WiFi.h>
#include <ESP_Mail_Client.h> // ESP-Mail-Client 라이브러리

// WiFi 네트워크 정보
const char* ssid = "your_wifi_ssid";         // WiFi 이름
const char* password = "your_wifi_password"; // WiFi 비밀번호

// SMTP 서버 정보
#define SMTP_HOST "smtp.gmail.com" // Gmail SMTP 서버
#define SMTP_PORT 587              // STARTTLS 포트 번호

// 이메일 계정 정보
#define AUTHOR_EMAIL "your_email@gmail.com"   // 발신자 이메일
#define AUTHOR_PASSWORD "your_password"       // 발신자 비밀번호 (또는 앱 비밀번호)

// 수신자 이메일 주소
#define RECIPIENT_EMAIL "recipient_email@gmail.com"

// SMTP 세션 및 메시지
SMTPSession smtp;
SMTP_Message message;

void setup() {
  Serial.begin(115200);

  // WiFi 연결
  Serial.print("Connecting to WiFi");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected!");
  Serial.println("IP address: " + WiFi.localIP().toString());

  // SMTP 세션 설정
  ESP_Mail_Session session;
  session.server.host_name = SMTP_HOST;
  session.server.port = SMTP_PORT;
  session.login.email = AUTHOR_EMAIL;
  session.login.password = AUTHOR_PASSWORD;
  session.login.user_domain = ""; // 필요시 도메인 설정

  // 이메일 메시지 구성
  message.sender.name = "ESP32 Test Device"; // 발신자 이름
  message.sender.email = AUTHOR_EMAIL;      // 발신자 이메일
  message.subject = "Hello from ESP32!";    // 이메일 제목
  message.addRecipient("Recipient", RECIPIENT_EMAIL); // 수신자
  message.text.content = "This is a test email sent from ESP32."; // 본문
  message.text.charSet = "utf-8"; // 문자 인코딩 설정

  // 이메일 전송
  if (!MailClient.sendMail(&smtp, &session, &message)) {
    Serial.println("Failed to send email");
    Serial.println("Error: " + smtp.errorReason());
  } else {
    Serial.println("Email sent successfully!");
  }

  // 연결 종료
  smtp.closeSession();
}

void loop() {
  // 메인 루프는 비워 둡니다 (필요 시 다른 작업 추가 가능)
}




void setup() {
  Serial.begin(9600); // 시리얼 통신 시작
}

void loop() {
  int co2Level = analogRead(A0); // 이산화탄소 센서 값 읽기
  Serial.println(co2Level); // Jetson Nano로 데이터 전송
  delay(1000); // 1초 대기
}


