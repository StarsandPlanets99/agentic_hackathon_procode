# 🛍️ AI Shopping Agents with AutoGen

This hackathon project demonstrates how to build an **agentic AI backend** using Microsoft's **AutoGen** framework. The AI agent uses the **WebSurfer** (Playwright-based) to browse the web in a visible browser window and perform tasks like **adding items to an online shopping basket**.

---

## 📌 Project Objective

Use a GPT powered autonomous agent to:
- Navigate an e-commerce site
- Search for specific items
- Add selected items to the shopping basket

---

## 🧰 Prerequisites

Before you begin, make sure you have the following installed:

### ✅ Python 3.10 or later
Download Here: https://www.python.org/downloads/

### ✅ Node.js
Required for Playwright browser automation  
Download: https://nodejs.org/

### ✅ Azure OpenAI API Key

# ⚙️ Azure OpenAI Setup Guide 

This guide walks you through how to set up your Azure tenant, create an Azure OpenAI resource, deploy a GPT model (e.g GPT-4o), and retrieve your credentials for integration.

---

Before starting, ensure you have:

- ✅ An Active **Azure Subscription**
- ✅ Access to **Azure OpenAI Service** 
- ✅ Azure Role Assignment: `Cognitive Services Contributor` or `Cognitive Services OpenAI Contributor` --- check this

---

## Step 1: Create an Azure OpenAI Resource

1. **Sign in** to [https://portal.azure.com](https://portal.azure.com)
2. Click **"Create a resource"**
3. Search for **"Azure OpenAI"** and select it
4. Click **"Create"**

### Fill in the following:

- **Subscription**: Your active subscription  
- **Resource Group**: Create or use an existing one (e.g. `hackathon-resource-group`)  
- **Region**: Choose a region with GPT-4o (e.g. `UK South`)  
- **Name**: Unique name like `openai-resource`  
- **Pricing Tier**: `Standard S0`  

### Network Settings:

- Use **"All networks"** for public access (or configure private access later)

5. Click **"Review + Create"** → then **"Create"**

---

## Step 2: Deploy a Model

1. Go to [Azure OpenAI Studio](https://oai.azure.com/)
2. Select your **subscription** and **resource**
3. On the left sidebar, go to **Deployments**
4. Click **"Create new deployment"**

### Configure the deployment:

- **Model**: Choose `gpt-4o` (or other available models)
- **Deployment name**: e.g. `gpt4o-deployment`
- **Deployment type**: Use `Standard` for most cases

5. Click **"Create"**

---

## Step 3: Get API Keys and Endpoint

1. In the [Azure Portal](https://portal.azure.com), go to your **Azure OpenAI resource**
2. Under **Keys and Endpoint**, you’ll find:
   - **Endpoint** (e.g. `https://my-openai-resource.openai.azure.com/`)
   - **API keys**: Two interchangeable keys

> ⚠️ **Keep your API keys secret**. Do not commit them to GitHub!

---

## Step 4: Test the Deployment

### Via Python

```python
import openai

openai.api_type = "azure"
openai.api_base = "https://YOUR-RESOURCE-NAME.openai.azure.com/"
openai.api_version = "2024-12-01-preview"
openai.api_key = "YOUR-API-KEY"

response = openai.ChatCompletion.create(
    engine="YOUR-DEPLOYMENT-NAME",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)

print(response.choices[0].message["content"])
```

---

## ⚙️ Setup Instructions

### 1. Clone the Repo

```bash
git clone 
cd ai-shopping-agent
```

### 2. Create and Activate a Virtual Environment

```bash
python3 -m venv autogen_env
source autogen_env/bin/activate  # On Windows: autogen_env\Scripts\activate
```

### 3. Install Dependencies
```bash
pip install -U autogen-agentchat autogen-ext[openai,web-surfer]
playwright install
```
>💡 playwright is required to download browser binaries

### 4. Use a .env File for Environment Variables

Create a .env file (and .gitignore file) in the project root

```env
OPENAI_API_KEY=your-api-key
OPENAI_API_TYPE=azure
OPENAI_API_BASE=https://your-resource-name.openai.azure.com/
OPENAI_API_VERSION=2024-12-01-preview
```
> ⚠️ **Add .env to your .gitignore file**. Do not commit it to GitHub:
```bash
.env
```

---

## 🚀 Run the Agents

Create a script called e.g shopping_agents.py and paste the following:
 #### edit this 
```python
#IMPORTS
import asyncio
from autogen_agentchat.agents import UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.agents.web_surfer import MultimodalWebSurfer
```

async def main():
    model_client = OpenAIChatCompletionClient(model="gpt-4o")

    web_surfer = MultimodalWebSurfer(
        name="web_surfer",
        model_client=model_client,
        headless=False,           # Enables visible browser window
        animate_actions=True
    )

    user_proxy = UserProxyAgent(name="user_proxy")

    team = RoundRobinGroupChat(
        agents=[web_surfer, user_proxy],
        termination_condition=TextMentionTermination("exit", sources=["user_proxy"])
    )

    try:
        await Console(team.run_stream(
            task="Browse an e-commerce site and add a specific item to the shopping basket."
        ))
    finally:
        await web_surfer.close()
        await model_client.close()

asyncio.run(main())

```
EXTENSION TASKS:
- explain teams
- Use different types of teams
- use assistant agents to create a multi agent system
- task (add specific items to shopping basket)

