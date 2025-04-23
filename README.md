# weather-bot-project
Local ADK Weather Bot Tutorial (macOS / Terminal / VS Code)

This guide converts the progressive Weather Bot tutorial into a local project structure suitable for running on macOS using Terminal (or VS Code's integrated terminal) and VS Code as your editor. We'll leverage the ADK Dev UI (adk web) and follow best practices.

**Prerequisites**:

- Python 3.9+ installed
- Visual Studio Code (or another code editor)
- API Keys for desired LLMs (Gemini, OpenAI, Anthropic)


**Phase 1: Project Setup and Environment**

**1. Create Project Directory:**

```bash
# Choose a location for your projects, e.g., ~/Projects
# cd ~/Projects
mkdir weather_bot_project
cd weather_bot_project
```

**2. Set Up Virtual Environment:**

This creates an isolated space for your project's Python packages.

```bash
# Create the virtual environment folder named .venv
python3 -m venv .venv

# Activate the environment (run this command *each time* you open a new terminal for this project)
source .venv/bin/activate
```

Your terminal prompt should now start with (.venv).

**3. Open Project in VS Code:**

(Optional, but recommended)

```bash
code .
```

This opens the weather_bot_project folder in VS Code. You can use VS Code's integrated terminal (Terminal > New Terminal) for the subsequent commands, and it should automatically use the activated .venv if you opened it after activation.

**4. Create requirements.txt:**
In VS Code or using touch, create a file named requirements.txt in the weather_bot_project directory. Add the following lines:

```plaintext
# weather_bot_project/requirements.txt
google-adk
litellm
python-dotenv # Useful for loading .env file if running scripts directly
```

**5. Install Dependencies:**

In your activated terminal:

```bash
pip install -r requirements.txt
```

**6. Create Agent Package Directory:**

This directory will hold your specific agent code.

```bash
mkdir weather_bot_team
```

**7. Create .gitignore:**

Create a file named .gitignore in the weather_bot_project directory. This tells Git version control which files to ignore.

```
# weather_bot_project/.gitignore
.venv/
__pycache__/
*.pyc
*.pyo
*.pyd
.DS_Store

# Ignore environment files containing secrets
.env
*.env
```

**Phase 2: Project Structure Overview**

Before creating the code files, here is the target structure you will build:

```plaintext
weather_bot_project/          <- Your main project folder
│
├── .venv/                     # Python virtual environment (managed by python)
│   └── ...
│
├── weather_bot_team/          # Python package containing your agent code
│   ├── __init__.py            # Makes it a package, exposes agent(s)
│   ├── agents.py              # Defines Agent instances (root, sub-agents)
│   ├── tools.py               # Defines agent tool functions
│   ├── callbacks.py           # Defines guardrail callback functions
│   └── .env                   # Stores API keys (IMPORTANT!)
│
├── requirements.txt           # Lists Python dependencies
└── .gitignore                 # Specifies files/folders for Git to ignore
```

**Phase 3: Create Code Files**

Now, create the Python files inside the weather_bot_team directory using VS Code's file explorer or the touch command in the terminal (e.g., touch weather_bot_team/__init__.py). Populate them with the code provided below.

1. weather_bot_team/__init__.py:

- Purpose: Makes weather_bot_team a Python package and exposes the main agent instance for the ADK Dev UI.

- Content:
```python
# weather_bot_project/weather_bot_team/__init__.py

# Import and expose the final root agent instance
# ADK CLI tools like 'adk web' will look for Agent instances imported here.
from .agents import root_agent_tool_guardrail

print("Weather Bot Team package initialized.")
```

2. weather_bot_team/.env:

- Purpose: Stores your API keys securely inside the agent package directory.
- Action: Create this file.
- Content: Replace the placeholder values with your actual API keys!
```
# weather_bot_project/weather_bot_team/.env

# Set to False to use API Keys directly (required for LiteLLM multi-model setup)
GOOGLE_GENAI_USE_VERTEXAI=False

# Gemini API Key (Get from Google AI Studio: https://aistudio.google.com/app/apikey)
GOOGLE_API_KEY="YOUR_GOOGLE_API_KEY" # <--- REPLACE

# OpenAI API Key (Get from OpenAI Platform: https://platform.openai.com/api-keys)
OPENAI_API_KEY="YOUR_OPENAI_API_KEY" # <--- REPLACE

# Anthropic API Key (Get from Anthropic Console: https://console.anthropic.com/settings/keys)
ANTHROPIC_API_KEY="YOUR_ANTHROPIC_API_KEY" # <--- REPLACE
```

- Security Note: Ensure this .env file is listed in your .gitignore (it is) and is never committed to version control.

3. weather_bot_team/tools.py:
- Purpose: Contains all the tool functions the agents can use.
- Content: (Copy the full content from the previous response's Step 3 - tools.py)
```python
# weather_bot_project/weather_bot_team/tools.py
import logging
from google.adk.tools.tool_context import ToolContext

# Basic Logging for Tool Calls
logging.basicConfig(level=logging.INFO) # You can adjust level
log = logging.getLogger(__name__)

# --- Tool from Step 1 (Used by early agent versions, kept for reference if needed) ---
def get_weather(city: str) -> dict:
    """Retrieves the current weather report for a specified city (MOCK). ... """
    log.info(f"--- Tool: get_weather called for city: {city} ---")
    # ... (rest of get_weather function code) ...
    city_normalized = city.lower().replace(" ", "")
    mock_weather_db = {
        "newyork": {"status": "success", "report": "The weather in New York is sunny with a temperature of 25°C."},
        "london": {"status": "success", "report": "It's cloudy in London with a temperature of 15°C."},
        "tokyo": {"status": "success", "report": "Tokyo is experiencing light rain and a temperature of 18°C."},
    }
    if city_normalized in mock_weather_db: return mock_weather_db[city_normalized]
    else: return {"status": "error", "error_message": f"Sorry, I don't have weather information for '{city}'."}

# --- Tools from Step 3 (Greeting/Farewell) ---
def say_hello(name: str = "there") -> str:
    """Provides a simple greeting, optionally addressing the user by name. ... """
    log.info(f"--- Tool: say_hello called with name: {name} ---")
    # ... (rest of say_hello function code) ...
    return f"Hello, {name}!"

def say_goodbye() -> str:
    """Provides a simple farewell message to conclude the conversation."""
    log.info(f"--- Tool: say_goodbye called ---")
    # ... (rest of say_goodbye function code) ...
    return "Goodbye! Have a great day."

# --- Tool from Step 4 (Stateful Weather) ---
def get_weather_stateful(city: str, tool_context: ToolContext) -> dict:
    """Retrieves weather, converts temp unit based on session state (MOCK). ... """
    log.info(f"--- Tool: get_weather_stateful called for {city} ---")
    # ... (rest of get_weather_stateful function code) ...
    preferred_unit = tool_context.state.get("user_preference_temperature_unit", "Celsius")
    log.info(f"--- Tool: Reading state 'user_preference_temperature_unit': {preferred_unit} ---")
    city_normalized = city.lower().replace(" ", "")
    mock_weather_db = { "newyork": {"temp_c": 25, "condition": "sunny"}, "london": {"temp_c": 15, "condition": "cloudy"}, "tokyo": {"temp_c": 18, "condition": "light rain"}, }
    if city_normalized in mock_weather_db:
        data = mock_weather_db[city_normalized]
        temp_c, condition = data["temp_c"], data["condition"]
        if preferred_unit.lower() == "fahrenheit": temp_value, temp_unit = (temp_c * 9/5) + 32, "°F"
        else: temp_value, temp_unit = temp_c, "°C"
        report = f"The weather in {city.capitalize()} is {condition} with a temperature of {temp_value:.0f}{temp_unit}."
        result = {"status": "success", "report": report}
        log.info(f"--- Tool: Generated report in {preferred_unit}. Result: {result} ---")
        tool_context.state["last_city_checked_stateful"] = city
        log.info(f"--- Tool: Updated state 'last_city_checked_stateful': {city} ---")
        return result
    else:
        error_msg = f"Sorry, I don't have weather information for '{city}'."
        log.info(f"--- Tool: City '{city}' not found. ---")
        return {"status": "error", "error_message": error_msg}

print("Weather Bot Tools defined.")
```

4. weather_bot_team/callbacks.py:
- Purpose: Contains the guardrail callback functions.
- Content: (Copy the full content from the previous response's Step 4 - callbacks.py)
```python
# weather_bot_project/weather_bot_team/callbacks.py
import logging
from google.adk.agents.callback_context import CallbackContext
# ... other necessary imports ...
from google.adk.models.llm_request import LlmRequest
from google.adk.models.llm_response import LlmResponse
from google.genai import types
from google.adk.tools.base_tool import BaseTool
from google.adk.tools.tool_context import ToolContext
from typing import Optional, Dict, Any

logging.basicConfig(level=logging.INFO)
log = logging.getLogger(__name__)

# --- Callback from Step 5 (Input Guardrail) ---
def block_keyword_guardrail( # ... function signature ...
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmResponse]:
    """ Inspects the latest user message for 'BLOCK'. ... """
    # ... (rest of block_keyword_guardrail function code) ...
    agent_name = callback_context.agent_name
    log.info(f"--- Callback: block_keyword_guardrail running for agent: {agent_name} ---")
    last_user_message_text = "" # ... logic to extract last user message ...
    if llm_request.contents: # ... (implementation as before) ...
        for content in reversed(llm_request.contents):
            if content.role == 'user' and content.parts:
                if content.parts[0].text: last_user_message_text = content.parts[0].text; break
    log.debug(f"--- Callback: Inspecting last user message: '{last_user_message_text[:100]}...' ---")
    keyword_to_block = "BLOCK"
    if keyword_to_block in last_user_message_text.upper(): # ... (blocking logic as before) ...
        log.warning(f"--- Callback: Found '{keyword_to_block}'. Blocking LLM call for {agent_name}! ---")
        callback_context.state["guardrail_block_keyword_triggered"] = True; log.info(f"--- Callback: Set state 'guardrail_block_keyword_triggered': True ---")
        return LlmResponse(content=types.Content(role="model", parts=[types.Part(text=f"I cannot process this request because it contains the blocked keyword '{keyword_to_block}'.")]))
    else: log.debug(f"--- Callback: Keyword not found. Allowing LLM call for {agent_name}. ---"); return None

# --- Callback from Step 6 (Tool Guardrail) ---
def block_paris_tool_guardrail( # ... function signature ...
    tool: BaseTool, args: Dict[str, Any], tool_context: ToolContext
) -> Optional[Dict]:
    """ Checks if 'get_weather_stateful' is called for 'Paris'. ... """
    # ... (rest of block_paris_tool_guardrail function code) ...
    tool_name, agent_name = tool.name, tool_context.agent_name
    log.info(f"--- Callback: block_paris_tool_guardrail running for tool '{tool_name}' in agent '{agent_name}' ---"); log.debug(f"--- Callback: Inspecting args: {args} ---")
    target_tool_name, blocked_city = "get_weather_stateful", "paris"
    if tool_name == target_tool_name: # ... (blocking logic for Paris as before) ...
         city_argument = args.get("city", "")
         if city_argument and city_argument.lower() == blocked_city:
             log.warning(f"--- Callback: Detected blocked city '{city_argument}' for tool '{tool_name}'. Blocking tool execution! ---")
             tool_context.state["guardrail_tool_block_triggered"] = True; log.info(f"--- Callback: Set state 'guardrail_tool_block_triggered': True ---")
             return {"status": "error", "error_message": f"Policy restriction: Weather checks for '{city_argument.capitalize()}' are currently disabled by a tool guardrail."}
         else: log.debug(f"--- Callback: City '{city_argument}' is allowed for tool '{tool_name}'. ---")
    else: log.debug(f"--- Callback: Tool '{tool_name}' is not the target tool. Allowing. ---")
    log.debug(f"--- Callback: Allowing tool '{tool_name}' to proceed. ---"); return None


print("Weather Bot Callbacks defined.")
```
5. weather_bot_team/agents.py:
- Purpose: Defines the agents, including the final root agent using the tools and callbacks.
- Content: (Copy the full content from the previous response's Step 5 - agents.py)
```python
# weather_bot_project/weather_bot_team/agents.py
import os
from google.adk.agents import Agent
# ... other necessary imports ...
from google.adk.models.lite_llm import LiteLlm
from .tools import get_weather_stateful, say_hello, say_goodbye, get_weather
from .callbacks import block_keyword_guardrail, block_paris_tool_guardrail

# --- Define Model Constants ---
MODEL_GEMINI_2_0_FLASH = "gemini-2.0-flash" # ... etc ...
MODEL_GPT_4O = "openai/gpt-4o"
MODEL_CLAUDE_SONNET = "anthropic/claude-3-sonnet-20240229"

# --- Sub-Agents ---
greeting_agent = None # Initialize
try: # ... (definition of greeting_agent as before) ...
    greeting_agent = Agent( model=MODEL_GEMINI_2_0_FLASH, name="greeting_agent", instruction="...", description="...", tools=[say_hello], )
except Exception as e: print(f"Warning: Could not create Greeting agent. Error: {e}")

farewell_agent = None # Initialize
try: # ... (definition of farewell_agent as before) ...
     farewell_agent = Agent( model=MODEL_GEMINI_2_0_FLASH, name="farewell_agent", instruction="...", description="...", tools=[say_goodbye], )
except Exception as e: print(f"Warning: Could not create Farewell agent. Error: {e}")

# --- Final Root Agent ---
root_agent_tool_guardrail = None # Initialize
if greeting_agent and farewell_agent:
    try: # ... (definition of root_agent_tool_guardrail as before) ...
         root_agent_tool_guardrail = Agent(
             name="weather_agent_v6_tool_guardrail", model=MODEL_GEMINI_2_0_FLASH, description="...",
             instruction="You are the main Weather Agent coordinating a team. ... Inform the user if a tool returns an error or if a guardrail blocks an action (like checking weather for Paris or using the word BLOCK).",
             tools=[get_weather_stateful], sub_agents=[greeting_agent, farewell_agent], output_key="last_weather_report",
             before_model_callback=block_keyword_guardrail, before_tool_callback=block_paris_tool_guardrail )
         print(f"✅ Final Root Agent '{root_agent_tool_guardrail.name}' created.")
    except Exception as e: print(f"❌ FATAL: Could not create the final Root Agent. Error: {e}")
else: print("❌ FATAL: Cannot create root agent because sub-agents failed.")

print("Weather Bot Agents defined.")
```

**Phase 4: Running the Agent with ADK Dev UI**

1. Navigate to Project Root:
Make sure your terminal's current directory is weather_bot_project (the folder containing .venv and weather_bot_team).
```bash
pwd # Should show path ending in /weather_bot_project
```
2. Ensure Virtual Environment is Active:

Confirm your prompt starts with (.venv). If not, run source .venv/bin/activate.

3. Run the ADK Dev UI:

```bash
adk web
```

4. Access the Dev UI:

- The terminal will show output as ADK loads agents and configurations. It should mention loading keys from the .env file.
- Open the URL provided (usually http://localhost:8000) in your web browser.

5. Interact:

- In the Dev UI, select the weather_agent_v6_tool_guardrail agent from the dropdown.
- Use the chat interface to test the different functionalities: greetings, farewells, allowed weather requests (e.g., "What is the weather in London?"), blocked weather requests ("How about Paris?"), and input guardrail triggers ("BLOCK this").
- Inspect the interaction details on the right panel.
---
This updated guide provides a complete workflow for setting up and running the advanced Weather Bot agent team locally on your Mac using Terminal and VS Code, with the project structure clearly laid out. Remember to handle your API keys securely in the .env file.
