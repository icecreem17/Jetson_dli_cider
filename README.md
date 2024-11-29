# jetson_dli_cider

잭슨 나노 일대기
===============
import serial
import pandas as pd
import time
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.base import MIMEBase
from email import encoders

# 시리얼 포트 설정 (Jetson Nano의 UART 포트 사용)
ser = serial.Serial('/dev/ttyUSB0', 9600)  # 포트를 Arduino와 연결된 것으로 변경
data_list = []

# 이메일 전송 함수
def send_email(file_path):
    # 이메일 설정
    sender_email = "your_email@gmail.com"
    receiver_email = "recipient_email@gmail.com"
    password = "your_app_password"

    # 이메일 메시지 생성
    msg = MIMEMultipart()
    msg['From'] = sender_email
    msg['To'] = receiver_email
    msg['Subject'] = "Sensor Data (CO2, Temperature, Humidity)"

    # 파일 첨부
    attachment = MIMEBase('application', 'octet-stream')
    with open(file_path, "rb") as file:
        attachment.set_payload(file.read())
    encoders.encode_base64(attachment)
    attachment.add_header('Content-Disposition', f'attachment; filename={file_path}')
    msg.attach(attachment)

    # 이메일 전송
    with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
        server.login(sender_email, password)
        server.sendmail(sender_email, receiver_email, msg.as_string())

    print(f"Email sent to {receiver_email} with {file_path} attached.")

# 데이터 수집 및 저장
def collect_and_save_data():
    start_time = time.time()  # 시작 시간 기록
    formatted_start_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(start_time))

    try:
        while time.time() - start_time < 30 * 60:  # 30분 동안 실행
            if ser.in_waiting > 0:
                # 시리얼에서 데이터 읽기
                raw_data = ser.readline().decode('utf-8').strip()
                
                # 데이터를 CO2, Temperature, Humidity로 분리
                try:
                    co2_level, temperature, humidity = raw_data.split(",")
                    timestamp = time.strftime("%Y-%m-%d %H:%M:%S")
                    data_list.append({
                        "Timestamp": timestamp,
                        "CO2_Level": co2_level,
                        "Temperature": temperature,
                        "Humidity": humidity
                    })
                    print(f"Received: CO2={co2_level}, Temp={temperature}, Humidity={humidity}")
                except ValueError:
                    print(f"Invalid data format: {raw_data}")
                    continue

        # 수집 종료 후 데이터 저장
        formatted_end_time = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
        # 시작 시간과 종료 시간 추가
        data_list.append({"Timestamp": "Start Time", "CO2_Level": "-", "Temperature": "-", "Humidity": formatted_start_time})
        data_list.append({"Timestamp": "End Time", "CO2_Level": "-", "Temperature": "-", "Humidity": formatted_end_time})
        
        # 데이터프레임 생성 및 엑셀 저장
        df = pd.DataFrame(data_list)
        file_path = "sensor_data.xlsx"
        df.to_excel(file_path, index=False)
        print(f"Excel file saved as '{file_path}'")
        
        # 이메일로 엑셀 파일 전송
        send_email(file_path)

    except KeyboardInterrupt:
        print("Data collection stopped by user.")

    finally:
        ser.close()

# 프로그램 실행
if __name__ == "__main__":
    collect_and_save_data()

day 1 : 
잭슨 나노 한글 설치까지
```

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


