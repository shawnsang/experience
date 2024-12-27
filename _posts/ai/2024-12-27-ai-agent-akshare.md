# AI 智能分析助手 - 基于 LLM 的 Akshare 数据分析工具

## 🎯 项目愿景

将复杂的金融数据分析变得简单！通过自然语言交互，让每个人都能轻松获取和分析金融数据。

---

### **背景**

`Akshare` https://akshare.akfamily.xyz/ 是一个功能强大的开源金融数据接口库，涵盖了股票、基金、期货等多个领域的数据信息。通过其接口，用户可以方便地获取实时市场数据和历史数据。为了简化用户查询流程，同时增强灵活性和智能化，我们希望利用 LLM（大语言模型）的 ReAct Agent 方法，自动解析 Akshare 的接口文档，实现自动化数据查询和智能推理。

通过引入 **ReAct (Reasoning and Acting)** 方法，我们可以将用户的自然语言问题拆解为逻辑推理、接口选择、参数填充以及数据查询操作，从而为用户提供高效且准确的查询结果。

---

### **系统目标**

- 解析 Akshare 的接口文档（股票、基金、期货等），生成多个 Agent，分别负责不同领域的数据查询。
  - 股票工具数量：310 (https://akshare.akfamily.xyz/data/stock/stock.html)
  - 期货工具数量：44 (https://akshare.akfamily.xyz/data/futures/futures.html) 
  - 公募基金工具数量：58 (https://akshare.akfamily.xyz/data/fund/fund_public.html)
  - 私募基金工具数量：14 (https://akshare.akfamily.xyz/data/fund/fund_private.html)
- 基于用户的自然语言输入，自动推理选择合适的 Agent 和接口方法。
- 自动生成查询代码，调用 Akshare 接口并返回结果。
- 提供易于扩展的 Python 实现，方便后续新增功能或接口。
- 支持中文、英文等输入。
- 模型增强：支持Ollama，Deepseek，Qwen，OpenAI 等模型。

---

### **用户体验示例**

```
🤖：我是智能金融分析助手，可以提供股票、基金、期货的信息，有什么可以帮您？
👨：中国银行最新的价格？
🤖：中国银行的最新股价为： 5.47

👨：公募基金中金额最大的是谁？
🤖：易方达基金管理有限公司是公募基金中金额最大的公司，其管理规模为 19940.44亿元。

👨：螺纹钢，现在是什么价格？
🤖：螺纹钢期货市场的当前价格： 3266.0， 最新日期： 2024-12-27
```

![gpt-4o-2024-08-06](/experience/assets/images/posts/ai/akshare/gpt-4o-2024-08-06.gif)

---

### **设计架构**

以下是系统的模块化设计：

```
用户输入 -> 自然语言理解 (LLM) -> 推理选择 (ReAct Agent)
		|
		+-> 接口解析器 (解析 Akshare 文档)
		|
		+-> 动态代码生成器 (生成查询代码)
		|
		+-> 查询执行器 (调用 Akshare 接口)
		|
		+-> 结果处理器 (格式化与返回结果)
```

---

### **功能扩展**

1. **用户交互**：集成到 Web 或 CLI 界面，使用绘图工具(如 Matplotlib)对行情数据进行可视化。
2. **Docker部署**：支持Docker部署，方便部署到服务器。

---

### **核心组件**

1. **接口解析器**
   
   - 将 Akshare 官方文档下载保存在本地，自动提取接口信息（包括功能、参数、示例等）。
2. **ReAct Agent 设计**
   
   - **Agent 池**：根据接口分类（股票、基金、期货等），创建多个 Agent，分别处理不同领域的查询请求。
   - **推理与选择**：通过 LLM 理解用户问题，并选择适当的 Agent 及接口。
   - **代码生成与执行**：根据选定的接口动态生成 Python 查询代码，并运行获取结果。
3. **查询与反馈**
   
   - 自动处理查询结果，包括格式化、过滤、聚合等。
   - 以表格或可视化形式返回查询结果，提升用户体验。

---

### **环境配置**

**已验证Python 版本 3.11**
建议使用conda 或者 venv 创建虚拟环境来运行，避免依赖包冲突。

#### 1. 克隆项目

```bash
# 克隆项目
git clone https://github.com/IndigoBlueInChina/Auto-GPT-Stock.git
cd Auto-GPT-Stock
```

#### 2. 创建虚拟环境

```bash
python -m venv venv
source venv/bin/activate  # Windows: .\venv\Scripts\activate
```

#### 3. 配置环境变量-LLM参数

首先创建并配置 `.env` 文件：

```bash
# 创建 .env 文件
cp demo.env .env

# 编辑 .env 文件，设置你的 OpenAI API Key
OPENAI_API_KEY=sk-your-api-key-here
```

#### 4. 安装依赖

依赖管理工具使用 `poetry`，安装依赖包，能够有效解决依赖包冲突问题。

```bash
pip install poetry
poetry install
```

#### 5. 运行程序

```bash
python main.py

🤖：我是智能金融分析助手，可以提供股票、基金、期货的信息，有什么可以帮您？
```

## 模型能力比较

| 模型\问题| 股票问题| 基金问题|期货问题|
| --- | --- | --- | --- |
| o1-mini-2024-09-12  | 多轮得出结果 | 没有得到结果 | 没有得到结果 |
| gpt-4o-2024-08-06 | 一轮得到结果 | 一轮得到结果 | 二轮得到结果 |
| claude-3-haiku-20240307 | 多轮未得到结果 | 出错  | NA |
| qwen2.5-32b-instruct | 多轮得到结果 | 多轮得到结果 | 找到工具，多轮没有得到结果 |


![o1-mini-2024-09-12](/experience/assets/images/posts/ai/akshare/o1-mini-2024-09-12.mp4)

![gpt-4o-2024-08-06](/experience/assets/images/posts/ai/akshare/gpt-4o-2024-08-06.mp4)

![claude-3-haiku-20240307](/experience/assets/images/posts/ai/akshare/claude-3-haiku-20240307.mp4)

![qwen2.5-32b-instruct](/experience/assets/images/posts/ai/akshare/qwen2.5-32b-instruct.mp4)