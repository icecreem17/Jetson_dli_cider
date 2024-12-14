### chatbot

```
import os
from openai import OpenAI
import json

os.environ['OPENAI_API_KEY'] = ''

OpenAI.api_key = os.getenv("OPENAI_API_KEY")

# **Agent 정의**
class Agent:
    def __init__(self, role, goal, function, tools=None, memory=True):
        self.role = role
        self.goal = goal
        self.function = function
        self.tools = tools or []
        self.memory = memory

    def execute(self, **kwargs):
        return self.function(**kwargs)


# **Task 정의**
class Task:
    def __init__(self, description, agent):
        self.description = description
        self.agent = agent

    def run(self, **kwargs):
        print(f"Executing Task: {self.description}")
        return self.agent.execute(**kwargs)

import Jetson.GPIO as GPIO
import time
import math
import serial
import time

def measure_co2():
    SERIAL_PORT = "/dev/ttyUSB0"  # 실제 연결된 포트로 변경
    BAUD_RATE = 9600

    try:
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=5) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(5)  # 응답 대기 시간

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
                print("Error: Unexpected response format.")
                return "Error: Unexpected response format."

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return "Error: Serial connection issue."

    except Exception as e:
        print(f"Error during measurement: {e}")
        return f"Error: {e}"


measure_co2_agent = Agent(
    role="Sensor",
    goal="현재 이산화탄소 농도를 측정합니다.",
    function=measure_co2,
)


# GPT 분석 함수
def gpt_analysis(co2_concentration):
    """
    GPT를 사용하여 CO2 농도를 분석하고 권고 사항을 생성합니다.
    """
    if isinstance(co2_concentration, int):

        messages = [
            {"role": "system", "content": "You are an expert in indoor air quality analysis."},
            {"role": "user", "content": f"The current CO2 concentration is {co2_concentration} ppm. "
                                         f"This is categorized based on indoor air quality standards. "
                                         "Provide a detailed explanation of the air quality, its health impacts, and the actions to take."},
        ]
        
        # prompt = f"""
        # The current CO2 concentration is {co2_concentration} ppm.
        # Based on this value:
        # - Analyze the indoor air quality.
        # - Explain the health immpact of this CO2 level.
        # - Recommend actions to improve air quality.
        # """
        try:
            client = OpenAI()
            response = client.Completion.create(
                model="text-davinci-003",
                messages=messages,
                max_tokens=300,
                temperature=0.7
            )
            return{
                "level":level,
                "advice": advice,
                "details": response["choices"][0]["text"].strip()
            }
            
        except Exception as e:
            return {"error": str(e)}
    else:
        return {"error" : f"Invalid CO2 value: {co2_concentration}"}


gpt_analysis_agent = Agent(
    role="Analyzer",
    goal="CO2 농도를 기반으로 GPT 모델을 사용해 실내 공기질을 3단계로 구분하고 환기 단계에 따른 권고 멘트를 생성합니다.",
    function=lambda co2_value: gpt_analysis(co2_value),
)

# **Task 정의**
co2_measure_task = Task(
    description="CO2 농도를 측정합니다.",
    agent=measure_co2_agent,
)

gpt_analysis_task = Task(
    description="측정된 CO2 농도를 기반으로 분석 결과를 생성합니다.",
    agent=gpt_analysis_agent,
)

# **Crew 정의**
class Crew:
    def __init__(self, tasks):
        self.tasks = tasks

    def run(self):
        context = {}
        for task in self.tasks:
            result = task.run(**context)
            if "error" in result:
                return result["error"]
            context.update(result)
        return context

# Crew 생성
co2_analysis_crew = Crew(tasks=[co2_measure_task, gpt_analysis_task])

# **Gradio를 통한 인터페이스**
def process_input(user_message):
    if "이산화탄소 농도를 알려줘" in user_message:
        result = co2_analysis_crew.run()
        if "error" in result:
            return f"Error: {result}"
        else:
            co2_value = result["co2_value"]
            analysis = result["analysis"]
            return f"현재 이산화탄소 농도는 {co2_value} ppm입니다.\n\n{analysis}"
    else:
        return "죄송합니다. 현재 이 질문에 대한 답변을 준비 중입니다."

def chatbot_interface():
    with gr.Blocks() as demo:
        gr.Markdown("# CO2 Ventilation Assistant")
        chatbot = gr.Chatbot(label="AI Chatbot for CO2 Analysis")
        user_textbox = gr.Textbox(label="Enter your message", placeholder="예: 이산화탄소 농도를 알려줘")
        user_textbox.submit(process_input, inputs=user_textbox, outputs=chatbot)
    demo.launch(share=True, debug=True)



use_functions = [
    {
        "type": "function",
        "function": {
            "name": "measure_co2",
            "description": "Reads CO2 concentration from a CM1106 sensor connected via serial port and returns the measured value in ppm.",
            "parameters": {
                "type": "object",
                "properties": {},
                "required": []
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "gpt_analysis",
            "description": "Analyzes CO2 data and determines air quality and ventilation requirements.",
            "parameters": {
                "type": "object",
                "properties": {
                    "co2_concentration": {
                        "type": "integer",
                        "description": "The measured CO2 concentration in ppm."
                    }
                },
                "required": ["co2_concentration"]
            }
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
            "gpt_analysis": gpt_analysis
        }

        messages.append(response_message)  # extend conversation with assistant's reply

        # Step 4: send the info for each function call and function response to GPT
        for tool_call in tool_calls:
            function_name = tool_call.function.name
            function_to_call = available_functions[function_name]
            function_args = json.loads(tool_call.function.arguments)


            print(function_args)

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

```
## measure_co2 메소드
```
def measure_co2():
    SERIAL_PORT = "/dev/ttyUSB0"  # 실제 연결된 포트
    BAUD_RATE = 9600

    try:
        with serial.Serial(SERIAL_PORT, BAUD_RATE, timeout=5) as ser:
            ser.write(b'\x11\x01\x01\xED')  # CM1106 센서 명령어
            time.sleep(5)  # 응답 대기 시간

            # 응답 데이터 읽기
            response = ser.read_all()
            print(f"Raw response (bytes): {response}")

            try:
                response = response.decode('ascii').strip()  # 응답을 ASCII로 디코딩
                print(f"Decoded response: {response}")
            except Exception as decode_error:
                print(f"Decoding error: {decode_error}")
                return {"error": "Failed to decode response."}

            # 데이터 형식 확인
            if response.startswith("CO2:"):
                co2_value = int(response.split(":")[1].strip())  # CO2 값을 정수로 변환
                return {"co2_value": co2_value}
            else:
                return {"error": "Unexpected response format."}

    except serial.SerialException as e:
        print(f"Serial connection error: {e}")
        return {"error": "Serial connection issue."}

    except Exception as e:
        print(f"Error during measurement: {e}")
        return {"error": str(e)}
```
## gpt_analysis 메소드
```
def gpt_analysis(co2_concentration):
    """
    GPT를 사용하여 CO2 농도를 분석하고 권고 사항을 생성합니다.
    """
    if isinstance(co2_concentration, int):
        messages = [
            {"role": "system", "content": "You are an expert in indoor air quality analysis."},
            {"role": "user", "content": f"The current CO2 concentration is {co2_concentration} ppm. "
                                         f"Analyze the air quality and provide recommendations."},
        ]
        try:
            client = OpenAI()
            response = client.Completion.create(
                model="text-davinci-003",
                messages=messages,
                max_tokens=300,
                temperature=0.7
            )
            return {
                "details": response["choices"][0]["text"].strip()
            }
        except Exception as e:
            return {"error": str(e)}
    else:
        return {"error": f"Invalid CO2 value: {co2_concentration}"}
```
## ask_openai 메소드
```
def ask_openai(llm_model, messages, user_message, functions=''):
    client = OpenAI()
    proc_messages = messages

    if user_message != '':
        proc_messages.append({"role": "user", "content": user_message})

    try:
        if functions == '':
            response = client.chat.completions.create(
                model=llm_model,
                messages=proc_messages,
                temperature=1.0
            )
        else:
            response = client.chat.completions.create(
                model=llm_model,
                messages=proc_messages,
                tools=functions,
                tool_choice="auto"
            )

        response_message = response.choices[0].message
        tool_calls = response_message.tool_calls

        if tool_calls:
            available_functions = {
                "measure_co2": measure_co2,
                "gpt_analysis": gpt_analysis
            }

            messages.append(response_message)  # 대화 확장

            for tool_call in tool_calls:
                function_name = tool_call.function.name
                function_to_call = available_functions[function_name]
                function_args = json.loads(tool_call.function.arguments)

                function_response = function_to_call(**function_args)

                proc_messages.append(
                    {
                        "tool_call_id": tool_call.id,
                        "role": "tool",
                        "name": function_name,
                        "content": function_response,
                    }
                )
            second_response = client.chat.completions.create(
                model=llm_model,
                messages=proc_messages,
            )
            assistant_message = second_response.choices[0].message.content
        else:
            assistant_message = response_message.content

        text = assistant_message.replace('\n', ' ').replace(' .', '.').strip()
        proc_messages.append({"role": "assistant", "content": assistant_message})

        return proc_messages, text

    except Exception as e:
        return proc_messages, str(e)
```
## Gradio 연동
```
def process_input(user_message):
    if "이산화탄소 농도를 알려줘" in user_message:
        result = co2_analysis_crew.run()
        if isinstance(result, str) and "error" in result:
            return f"Error: {result}"
        else:
            co2_value = result.get("co2_value")
            analysis = result.get("details", "No analysis available.")
            return f"현재 이산화탄소 농도는 {co2_value} ppm입니다.\n\n{analysis}"
    else:
        return "죄송합니다. 현재 이 질문에 대한 답변을 준비 중입니다."
```
## Gradio ui  구성
```
def process(user_message, chat_history):
    proc_messages, ai_message = ask_openai("text-davinci-003", messages, user_message, functions=use_functions)
    chat_history.append((user_message, ai_message))
    return "", chat_history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(label="채팅창")
    user_textbox = gr.Textbox(label="입력")
    user_textbox.submit(process, [user_textbox, chatbot], [user_textbox, chatbot])

demo.launch(share=True, debug=True)
```



