```
# 필요 패키지를 설치합니다. pyserial은 시리얼 통신을 위한 라이브러리입니다.
!pip install pyserial

# smbus는 I2C 통신을 지원하는 패키지입니다.
!pip install smbus

# openai 라이브러리를 최신 버전으로 업데이트하여 설치합니다.
!pip install -U openai

# 기본적인 Python 표준 라이브러리와 OpenAI API를 가져옵니다.
import os  # 환경 변수 관리
from openai import OpenAI  # OpenAI API 호출용
import json  # JSON 데이터 처리를 위한 모듈

# OpenAI API 키를 환경 변수에 저장합니다.
os.environ['OPENAI_API_KEY'] = ''  # API 키
OpenAI.api_key = os.getenv("OPENAI_API_KEY")  # 환경 변수에서 API 키를 가져옵니다.

# Jetson GPIO를 가져옵니다 (Jetson 보드에서 GPIO 제어를 위한 라이브러리).
import Jetson.GPIO as GPIO
import time  # 시간 제어를 위한 모듈
import math  # 수학 연산용 모듈
import serial  # 시리얼 통신 라이브러리

def measure_co2():
    SERIAL_PORT = "/dev/ttyUSB0"  # 실제 연결된 포트로 변경
    BAUD_RATE = 9600

    try:
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=10) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(10)  # 응답 대기 시간

            # 응답 데이터 읽기
            response = ser.read_all()  # 가능한 모든 데이터 읽기
            print(f"Raw response (bytes): {response}")

            try:
                response = response.decode('ascii')  # 디코딩 시도
                print(f"Decoded response: {response}")
            except Exception as decode_error:
                print(f"Decoding error: {decode_error}")
                return "Error: Failed to decode response."

            if response.startswith("CO2:"):
                co2_value = response.split(":")[1].strip()
                return co2_value
            else:
                print("response: " + response)
                print("Error: Unexpected response format.")
                return "Error: Unexpected response format."

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return "Error: Serial connection issue."

    except Exception as e:
        print(f"Error during measurement: {e}")
        return f"Error: {e}"

measure_co2()

# CO2 농도를 측정하고 환기 필요 여부를 판단하는 함수
def determine_ventilation_status(co2_value):
    try:
        co2_value = int(co2_value)  # 사용자가 제공한 CO2 값을 정수로 변환
    except ValueError:
        return "올바른 이산화탄소 농도를 입력해 주세요. 예: 900ppm"

    # CO2 농도 범위에 따른 환기 판단
    if co2_value < 800:
        status = "농도가 적정 수준입니다. 환기가 필요하지 않습니다."
    elif co2_value <= 1000:
        status = "농도가 다소 높은 상태입니다. 환기를 권장합니다."
    else:
        status = "농도가 높아 공기질이 나쁩니다. 즉시 환기가 필요합니다."

    # 최종 메시지 반환
    return f"현재 이산화탄소 농도는 {co2_value}ppm입니다. {status}"

# OpenAI와의 상호작용에서 CO2 상태를 판단하도록 함수 목록에 추가
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
            "name": "determine_ventilation_status",  # 함수 이름
            "description": "Determines ventilation needs based on CO2 concentration."
        }
    }
]

def ask_openai(llm_model, messages, user_message, functions = ''):
    client = OpenAI()
    proc_messages = messages

    if user_message != '':
        proc_messages.append({"role": "user", "content": user_message})

    if functions == '':
        response = client.chat.completions.create(model=llm_model, messages=proc_messages, temperature = 1.0)
    else:
        response = client.chat.completions.create(model=llm_model, messages=proc_messages, tools=functions, tool_choice="auto") # 이전 코드와 바뀐 부분

    response_message = response.choices[0].message
    tool_calls = response_message.tool_calls

    if tool_calls:
        # Step 3: call the function
        # Note: the JSON response may not always be valid; be sure to handle errors

        available_functions = {
            "measure_co2": measure_co2,
            "determine_ventilation_status": determine_ventilation_status
        }

        messages.append(response_message)  # extend conversation with assistant's reply

        # Step 4: send the info for each function call and function response to GPT
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)


            
            print(function_args)
           

            if function_name == "measure_co2":
                co2_result= function_to_call(**function_args)
                
                ventilation_result = available_functions["determine_ventilation_status"](co2_result)

                proc_messages.append(
                    {
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": "determine_ventilation_status",
                        "content": ventilation_result,
                    }
                )
                
            else:
                
                if 'user_prompt' in function_args:
                    function_response = function_to_call(function_args.get('user_prompt'))
                else:
                    function_response = function_to_call(**function_args)
    
    
                proc_messages.append(
                    {
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": function_response,
                    }
                )  # extend conversation with function response

                
        second_response = client.chat.completions.create(
            model=llm_model,
            messages=messages,
        )  # get a new response from GPT where it can see the function response

        assistant_message = second_response.choices[0].message.content
    else:
        assistant_message = response_message.content

    text = assistant_message.replace('\n', ' ').replace(' .', '.').strip()


    proc_messages.append({"role": "assistant", "content": assistant_message})

    return proc_messages, text

import gradio as gr
import random


messages = []

def process(user_message, chat_history):

    # def ask_openai(llm_model, messages, user_message, functions = ''):
    proc_messages, ai_message = ask_openai("gpt-4o-mini", messages, user_message, functions= use_functions)

    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="채팅창")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)

