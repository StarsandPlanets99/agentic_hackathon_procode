#### 01 Introduction

### 🧠 How It Works

This project uses **AutoGen**, a powerful open-source framework from Microsoft that lets multiple AI "agents" talk to each other and work together to solve complex tasks. Here's a breakdown of the main building blocks involved:

---

### 🤖 What Are Agents?

Think of **agents** like intelligent team members. Each one has a specific role.

In this project:

- **WebSurfer Agent**: It can open real websites, click buttons, type in search boxes, and interact like a real user.
- **UserProxy Agent**: Represents *you*. It acts as your voice in the system and lets you send or receive messages in the agent conversation.

Agents talk to each other using natural language (text), and AutoGen coordinates this as a chat.

---

### 🧰 What Are Tools?

Some agents can use **tools** – code that help them perform specific actions such as:

- Search a database
- Call an API
- Extract info from a document

> 💡 In this project we will not be building any custom tools, instead the **WebSurfer** already has a built-in "tool": it controls a real browser using a system called **Playwright**. This lets it click buttons, navigate websites, and simulate actions like a human user would.

---

### 🧠 What Is Memory?

**Memory** in AutoGen allows agents to **remember** what happened earlier. This is useful for:

- Keeping track of what items were already viewed or added
- Referencing earlier steps
- Maintaining conversation history

> 💡 This project basic doesn’t use memory — but you can add it later.

---

### 🌐 What Is the WebSurfer?

The **MultimodalWebSurfer** is a special agent that can:

- Open real websites in a visible browser window (not just simulate clicks invisibly)
- See and interact with what’s on the page
- Make decisions (e.g what to click, where to type, etc.), using an LLM such as GPT-4o

It uses a tool called **Playwright** under to control the browser, and GPT-4o to decide what actions to take.

---

### 🧑‍🤝‍🧑 How Are Agents Organised? (Team Setup)

Agents don’t work alone, they’re grouped into **teams** that define how they interact. In AutoGen, teams are managed using **GroupChat types**.

For this hackathon, we’re keeping it simple with `RoundRobinGroupChat` and using this type to alternate between:
- The **WebSurfer**, who browses and clicks
- The **UserProxy**, who gives instructions and ends the chat

#### 🔄 `RoundRobinGroupChat`

This means agents take turns speaking one after the other: like a conversation where everyone gets a chance to respond.

> ℹ️ **Note:** AutoGen also supports other group chat types such as:
> - `SelectorGroupChat`: Chooses the most relevant agent to respond, instead of going in order.
> - `OrderedGroupChat`: Follows a strict sequence you define.
> - `MagenticOneGroupChat`: An advanced setup for coordinating complex, multi-agent workflows with tracing and logic.





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

### 🧩 How It All Comes Together

Once you have configured your agents:

1. The **task instruction** tells the team what to do (e.g. “Go to an e-commerce site and add headphones to the cart”)
2. The agents take turns thinking and acting
3. The **WebSurfer** opens the browser and performs the web navigation
4. The **UserProxyAgent** lets you observe or interact if needed
5. When done, you type `exit` to stop

---

```python
from autogen_agentchat.teams import RoundRobinGroupChat

team = RoundRobinGroupChat(
    agents=[web_surfer, user_proxy],
    termination_condition=TextMentionTermination("exit", sources=["user_proxy"])
)
```
