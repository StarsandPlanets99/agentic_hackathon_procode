## 🚀 Run the Agents

Create a script in the project root of your repository e.g ```shopping_agents.py``` and paste the following:

Add your imports:
```python
import asyncio
from autogen_agentchat.agents import UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.agents.web_surfer import MultimodalWebSurfer
```

Define your model:
```python

model_client = OpenAIChatCompletionClient(
     model="gpt-4o",
     api_key=api_key,
     api_base=api_base,
     api_version=api_version,
     api_type="azure"
)
```

Define your agents:
```python
#Web Surfer Agent
web_surfer = MultimodalWebSurfer(
     name="web_surfer",
     model_client=model_client,
     headless=False,           # Enables visible browser window
     animate_actions=True
 )

#User Proxy Agent
user_proxy = UserProxyAgent(name="user_proxy")
```
Create your team with the Round Robin Group Chat and Termination condition:
```python
team = RoundRobinGroupChat(
    agents=[web_surfer, user_proxy],
    termination_condition=TextMentionTermination("exit", sources=["user_proxy"])
)
```
> Here, the termination condition is triggered when the UserProxyAgent receives the text "exit" as input. This ends the session when the user decides to stop.

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

Extension tasks:
- Create a custom agent and add to your group chat
- Use another type of group chat
