# 🧠 Develop an AI Agent with Azure AI Foundry

In this exercise, you'll use **Azure AI Agent Service** to create a simple agent that analyzes data and creates charts using the built-in **Code Interpreter tool**.

> ⏱ Estimated Time: 30 minutes  
> ⚠️ Note: Some technologies used are in preview or active development. You may encounter unexpected behavior or errors.

---

## 🛠️ Prerequisites

- Azure subscription
- Access to [Azure AI Foundry Portal](https://ai.azure.com)
- Python environment (Cloud Shell or local)
- Familiarity with Python is helpful

---

## 🔹 Step 1: Create an Azure AI Foundry Project

1. Go to [https://ai.azure.com](https://ai.azure.com)  
2. Sign in with the following:
   - **Username**: ``
   - **Password**: ``
3. Close any welcome panels or Help panes.
4. On the home page, click **Create an agent**.
5. When prompted, enter:
   - **Project Name**: `Project54566302`
6. Expand **Advanced Options** and set:
   - **Azure AI Foundry resource**: `project54566302-resource`
   - **Subscription**: *(Your Azure subscription)*
   - **Resource Group**: `ResourceGroup1`
   - **Region**: Any **AI Foundry recommended** region

> ⚠️ Quotas may vary by region — switch regions if quota limits are hit.

7. Click **Create** and wait for the project to finish deploying.
8. Deploy a **gpt-4o model** when prompted (Global Standard or Standard).

> 🧠 A GPT-4o model may be auto-deployed when the agent/project is created.

9. Go to the **Overview** page and **copy the endpoint values** to a notepad. You’ll need them for the client app.

---

## 🔸 Step 2: Create an Agent Client App

### 📥 Clone the GitHub Repository

1. Open a new tab and sign into the [Azure Portal](https://portal.azure.com)
2. Use the `>` (Cloud Shell) button on the top bar.
3. Select **PowerShell**, and **switch to Classic version** (via Settings > "Go to Classic version").

4. Run the following commands in the shell:

```bash
rm -r ai-agents -f
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents ai-agents


cd ai-agents/Labfiles/02-build-ai-agent/Python
ls -a -l
```

## 🔧 Step 3: Configure the App

### 🧩 Install Dependencies

```bash
python -m venv labenv
./labenv/bin/Activate.ps1
pip install -r requirements.txt azure-ai-projects
code .env
```
Save with CTRL+S, close editor with CTRL+Q.

Replace your_project_endpoint with your actual project endpoint.

Set MODEL_DEPLOYMENT_NAME = gpt-4o

## 💻 Step 4: Write the Agent Code ##

```
code agent.py
```

### 🧱 Add Imports ###


```python
from azure.identity import DefaultAzureCredential
from azure.ai.agents import AgentsClient
from azure.ai.agents.models import FilePurpose, CodeInterpreterTool, ListSortOrder, MessageRole
```

### 🔌 Connect to Azure AI Agent ###

```python
# Connect to the Agent client
agent_client = AgentsClient(
   endpoint=project_endpoint,
   credential=DefaultAzureCredential(
       exclude_environment_credential=True,
       exclude_managed_identity_credential=True
   )
)
with agent_client:
```

### 📁 Upload Data & Create Code Interpreter Tool ###

```python
# Upload the data file and create a CodeInterpreterTool
file = agent_client.files.upload_and_poll(
    file_path=file_path, purpose=FilePurpose.AGENTS
)
print(f"Uploaded {file.filename}")

code_interpreter = CodeInterpreterTool(file_ids=[file.id])
```

### 🤖 Define the AI Agent ###

```python
# Define an agent that uses the CodeInterpreterTool
agent = agent_client.create_agent(
    model=model_deployment,
    name="data-agent",
    instructions="You are an AI agent that analyzes the data in the file that has been uploaded. Use Python to calculate statistical metrics as necessary.",
    tools=code_interpreter.definitions,
    tool_resources=code_interpreter.resources,
)
print(f"Using agent: {agent.name}")
```

### 🧵 Create a Chat Thread ###

```python
# Create a thread for the conversation
thread = agent_client.threads.create()
```

### 💬 Send User Prompt ###

```python
# Send a prompt to the agent
message = agent_client.messages.create(
    thread_id=thread.id,
    role="user",
    content=user_prompt,
)

run = agent_client.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
```

### ❌ Check Run Status ###

```python
# Check the run status for failures
if run.status == "failed":
    print(f"Run failed: {run.last_error}")
```

### 📩 Show Agent Response ###

```
# Show the latest response from the agent
last_msg = agent_client.messages.get_last_message_text_by_role(
   thread_id=thread.id,
   role=MessageRole.AGENT,
)
if last_msg:
   print(f"Last Message: {last_msg.text.value}")
```

### 📜 Display Conversation History ###

```
# Get the conversation history
print("\nConversation Log:\n")
messages = agent_client.messages.list(thread_id=thread.id, order=ListSortOrder.ASCENDING)
for message in messages:
   if message.text_messages:
       last_msg = message.text_messages[-1]
       print(f"{message.role}: {last_msg.text.value}\n")
```

### 🧹 Clean Up Resources ###

```
# Clean up
agent_client.delete_agent(agent.id)
```

### 🔐 Step 5: Sign Into Azure and Run the App ###

1. Authenticate with Azure:

```bash
az login
```

💡 You may need to specify a --tenant if using multiple tenants.

2. When prompted, sign in using:

Username: 

Password: 

3. Run the agent app
   
```bash
python agent.py
```
### 🔍 Step 6: Test the Agent ###

*Try prompts like:

📊 What's the category with the highest cost?

📈 Create a text-based bar chart showing cost by category

📐 What's the standard deviation of cost?

The conversation thread is stateful, so context is retained.

To exit the conversation, type: quit

### ✅ Summary ###

In this exercise, you:

* Created an Azure AI Agent Foundry project

* Used the Python SDK to build a client app

* Leveraged the Code Interpreter Tool to:

* Analyze data

* Generate charts

* Perform statistical calculations

* Built and tested a conversational agent that responds intelligently to user prompts
   



