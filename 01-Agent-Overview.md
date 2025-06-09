## 🚀 01 Agent Overview


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

> 💡 In this project, for simplicity we will not be using memory

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

For this hackathon, we will keep it simple with `RoundRobinGroupChat` and using this type to alternate between:
- The **WebSurfer**, who browses and clicks
- The **UserProxy**, who gives instructions and ends the chat

#### 🔄 `RoundRobinGroupChat`

This means agents take turns speaking one after the other: like a conversation where everyone gets a chance to respond.

> ℹ️ **Note:** AutoGen also supports other group chat types such as:
> - `SelectorGroupChat`: Chooses the most relevant agent to respond, instead of going in order.
> - `OrderedGroupChat`: Follows a strict sequence you define.
> - `MagenticOneGroupChat`: An advanced setup for coordinating complex, multi-agent workflows with tracing and logic.
