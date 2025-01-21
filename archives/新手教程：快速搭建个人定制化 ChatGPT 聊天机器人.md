对话机器人正变得越来越流行，它为用户提供了与技术互动的全新方式。借助 OpenAI 的 GPT 模型，开发者可以轻松创建功能强大的对话机器人。

在本教程中，我们将使用 Python 和 OpenAI API，在运行 Ubuntu 的 DigitalOcean Droplet 上构建并部署一个终端 ChatGPT 机器人。完成本教程后，你将拥有一个功能齐全的 GPT 机器人，可以直接从终端处理用户需求，提供实时互动的用户体验。无论你是经验丰富的开发者还是初学者，本教程都将帮助你掌握 ChatGPT 的应用能力，并构建属于你自己的定制 AI 机器人。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)

---

## 前提条件

在开始之前，请确保你具备以下条件：

- **云服务器**：一个至少有 4GB RAM 和 2 个 CPU 的云服务器。推荐使用 [DigitalOcean](https://bit.ly/bewildcard) 的 Droplet 云主机（新用户注册可享 200 美金免费额度，服务器最低 4 美元/月）。
- **操作系统**：Ubuntu 服务器，建议按照 [Ubuntu 初始服务器设置说明](https://bit.ly/bewildcard) 进行配置。
- **Python 环境**：在 Ubuntu Droplet 上安装 Python 3.7 或更高版本。
- **基础知识**：基本的 Python 编程知识。
- **OpenAI 账户**：一个具有 ChatGPT API 访问权限的 OpenAI 账户。

---

## 第一步 – 设置环境

在这一步中，我们将设置环境，以便在运行 Ubuntu 的 DigitalOcean Droplet 上构建并部署 ChatGPT 终端机器人。

### 创建 DigitalOcean Droplet

1. 登录你的 DigitalOcean 账户。
2. 进入 Droplets 部分。
3. 点击“Create Droplet”。
4. 选择 Ubuntu 操作系统（建议使用最新的 LTS 版本）。
5. 根据需求选择合适的计划。
6. 选择数据中心区域。
7. 添加 SSH 密钥以确保安全访问。
8. 点击“Create Droplet”。

### 连接到你的 Droplet

在本地计算机上打开终端，使用以下命令连接到 Droplet，替换 `<your_droplet_ip>` 为你的 Droplet 的 IP 地址：

bash
ssh root@<your_droplet_ip>


### 设置 Python 环境

1. 更新系统：

   bash
   sudo apt update
   sudo apt upgrade
   

2. 安装 Python 和 pip：

   bash
   sudo apt install python3 python3-pip
   

3. 安装 virtualenv：

   bash
   sudo pip3 install virtualenv
   

4. 创建项目文件夹并激活虚拟环境：

   bash
   mkdir my_chatgpt_bot
   cd my_chatgpt_bot
   virtualenv venv
   source venv/bin/activate
   

5. 安装所需的 Python 包：

   bash
   pip install openai textract
   

6. 配置 OpenAI API 密钥：

   - 获取你的 OpenAI API 密钥。
   - 打开 `.bashrc` 文件并添加以下内容：

     bash
     export OPENAI_API_KEY='your-api-key-here'
     

   - 重新加载环境变量：

     bash
     source ~/.bashrc
     

---

## 第二步 – 构建 ChatGPT 机器人

接下来，我们将编写 ChatGPT 机器人的代码。以下是主要步骤：

1. 创建一个名为 `mygptbot.py` 的文件：

   bash
   vi mygptbot.py
   

2. 复制以下代码到文件中：

   python
   import os
   import glob
   import openai
   import textract

   class Chatbot:
       def __init__(self):
           self.openai_api_key = os.getenv("OPENAI_API_KEY")
           self.chat_history = []

       def append_to_chat_history(self, message):
           self.chat_history.append(message)

       def read_personal_file(self, file_path):
           try:
               text = textract.process(file_path).decode("utf-8")
               return text
           except Exception as e:
               print(f"Error reading file {file_path}: {e}")
               return ""

       def collect_user_data(self):
           data_directory = "./data"
           data_files = glob.glob(os.path.join(data_directory, "*.*"))

           user_data = ""
           for file in data_files:
               file_extension = os.path.splitext(file)[1].lower()
               if file_extension in (".pdf", ".docx", ".xlsx", ".xls"):
                   user_data += self.read_personal_file(file)
               else:
                   with open(file, "r", encoding="utf-8") as f:
                       user_data += f.read() + "\n"
           return user_data

       def create_chat_response(self, message):
           self.append_to_chat_history(message)

           user_data = self.collect_user_data()
           messages = [
               {"role": "system", "content": "You are the most helpful assistant."},
               {"role": "user", "content": message},
           ]

           if user_data:
               messages.append({"role": "user", "content": user_data})

           response = openai.ChatCompletion.create(
               model="gpt-3.5-turbo",
               messages=messages,
               temperature=0.7,
               max_tokens=256,
               top_p=0.9,
               n=1,
               stop=None,
               frequency_penalty=0.9,
               presence_penalty=0.9
           )

           self.append_to_chat_history(response.choices[0].message.content.strip())
           return response.choices[0].message.content.strip()

       def start_chatting(self):
           while True:
               user_input = input("User: ")
               if user_input.lower() == "exit":
                   print("Chatbot: Goodbye! Have a great day!")
                   break
               bot_response = self.create_chat_response(user_input)
               print("Chatbot:", bot_response)

   chatbot = Chatbot()
   chatbot.start_chatting()
   

---

## 第三步 – 在终端运行 ChatGPT 机器人

1. 打开 Droplet 的控制台。
2. 运行以下命令启动机器人：

   bash
   python mygptbot.py
   

3. 现在，你可以与 ChatGPT 机器人进行交互。输入消息，机器人将根据输入进行回应。输入 `exit` 即可结束对话。

---

## 结论

通过本教程，你已经学习了如何使用 Python 在 Ubuntu 服务器上创建并部署一个功能强大的 ChatGPT 机器人。该机器人可以处理用户输入，并根据用户数据提供个性化的响应。你可以将其集成到其他平台，或进一步扩展为基于 Web 的聊天机器人。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bit.ly/bewildcard)