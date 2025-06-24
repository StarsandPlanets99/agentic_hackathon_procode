## 🚀 02 Running Agents

Create a script in the project root of your repository e.g ```shopping_agents.py``` and follow these steps:

1. Add your imports:
```python
import asyncio
import os
from dotenv import load_dotenv
from autogen_agentchat.agents import UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import AzureOpenAIChatCompletionClient
from autogen_ext.agents.web_surfer import MultimodalWebSurfer
```
2. Load your envrionment variables from your .env file:
```python
load_dotenv()
```
3. Create variables from your .env file:

```python
api_key = os.getenv("AZURE_OPENAI_API_KEY")
azure_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT")
api_version = os.getenv("AZURE_OPENAI_API_VERSION")
```

4. Define your model:
```python

model_client = AzureOpenAIChatCompletionClient(
     model="gpt-4o",
     api_key=api_key,
     azure_endpoint=azure_endpoint,
     azure_deployment="gpt-4o",
     api_version=api_version
)
```

5. Define your agents:
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
6. Create your team with the RoundRobinGroup Chat and set Termination condition:
```python
team = RoundRobinGroupChat(
    participants=[web_surfer, user_proxy],
    termination_condition=TextMentionTermination("exit", sources=["user_proxy"])
)
```
> The team uses RoundRobinGroupChat to alternate turns between the agents. The session ends when the user_proxy agent receives the word "exit" from your terminal input.

7. Define your main function with stream logic:
```python
async def main():
    # Start the task and stream responses to the terminal
    stream = team.run_stream(
        task="Browse the e-commerce site https://www.currys.co.uk/ and add headphones to the shopping basket."
    )
    
    # The Console streams the agent interactions to your terminal live
    await Console(stream)

    # Close connections after execution
    await web_surfer.close()
    await model_client.close()

```

8. Run the main function:
```python
if __name__ == "__main__":
    asyncio.run(main())
    print("🛍️ Shopping Complete: ✅")
```

### 🧩 How It All Comes Together

Once you have configured your agents, run your script in the terminal:

```python
python shopping_agents.py
```

A browser window should pop up, watch how the agent is navigating the web on its own. This process is fully autonomous - you don’t need to interact with the browser. Just observe as the agent explores the page. You may see a red dot moving around and clicking around the webpage, like a human would!

In the terminal, the Console class prints real-time updates of each agent's decisions and tool use, giving full observability into the task.
If you toggle between the web browser and the terminal you can see a full break down of what the agents are doing:

1. The **task instruction** tells the team what to do (e.g. “Go to an e-commerce site and add headphones to the cart”)
2. The agents take turns thinking and acting
3. The **WebSurfer** opens the browser and performs the web navigation
4. The **UserProxyAgent** lets you observe or interact if needed in the terminal
5. When done, you type `exit` to stop in the terminal

---

 > ⚠️ There may be times when agents fail to complete a task. Common reasons include:

- Security restrictions (e.g CAPTCHA, login prompts, bot detection)
- Incomplete instructions or vague goals
- Unsupported site layouts or dynamic JavaScript rendering

---

Designing reliable multi-agent systems is a real-world engineering challenge. Coordination often breaks down so building effective agentic workflows is an iterative process of trial, observation, and refinement.

### 💡 Creative Challenges (Optional Tasks)

These extra tasks help you explore AutoGen’s flexibility and push the limits of your web agent:


💭 Task 1:
- Did your agent manage to add items to the basket successfully?  
   If not, examine the terminal logs to understand what went wrong. Try:
  - Rephrasing the task more specifically (e.g "add the first item to the basket")
  - Asking the assistant to redirect to an alternative site
  - Adding intermediate steps (e.g “search first, then filter by price”)
 
    For example:
    ```python
      stream = team.run_stream(
        task="Browse the e-commerce site https://www.amazon.co.uk/ and add headphones to the shopping basket. Add the first item to the basket."
    )
    ```

💭 Task 2:
- Use the **UserProxyAgent** to interact mid-run:  
  Type guidance, clarifications, or commands directly into the terminal - you’re part of the loop!
<img width="550" alt="image" src="https://github.com/user-attachments/assets/9caa6aa4-93e4-44ad-afbd-9ac138b9739a" />


💭 Task 3:
- Create an additional custom **AssistantAgent** with a unique **system message**.
  For example, add an AssistantAgent with this behavior:
  “Instruct the WebSurfer agent to try alternative product pages if a site returns no results.”

  Try replacing the UserProxyAgent entirely with your custom AssistantAgent, to enable a more autonomous experience.
  
  You can read more about the AssistantAgent and different types of other agents here:
  🔗 [AutoGen Agent Guide](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/agents.html)

 > ⚠️ Don't forget to add any new agents you define to your group chat!
 > ⚠️ Don't forget to update your termination condition (or comment it out) if you are no longer using the UserProxyAgent for termination.


💭 Task 4:
- Try another group chat type like:
  - **SelectorGroupChat**: Automatically routes queries to the most relevant agent.

  You can read more about the different types of group chats here:
  🔗[AutoGen AgentChat Guide](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/index.html)
> You will find **SelectorGroupChat** under the Advanced section along with other types of chat

Have fun exploring AutoGen! It’s a powerful platform for building smart, collaborative agents that can read, search, click, and reason across the web!🌐

