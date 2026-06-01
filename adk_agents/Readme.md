# Google AI Development Kit (ADK) - Agent Design Patterns

This repository contains a comprehensive showcase of modern AI agent architectures and orchestration patterns built using the **Google AI Development Kit (ADK)** and the Gemini model family (e.g., `gemini-2.5-flash`). It serves as an educational and practical reference for developers building autonomous agents, multi-agent systems, and customized decision-making workflows.

---

## 📂 Repository Directory Structure

| Pattern Directory | Description | Core Features & Concepts | Main Files |
| :--- | :--- | :--- | :--- |
| [**a_single_agent**](file:///Users/manohar/ADK_Basic/a_single_agent) | Simple Single-Agent | Budget constraints, Google Search tool integration, location details. | [day_trip.py](file:///Users/manohar/ADK_Basic/a_single_agent/day_trip.py) |
| [**b1_sequential_agent**](file:///Users/manohar/ADK_Basic/b1_sequential_agent) | Linear Pipeline | `SequentialAgent`, state-sharing between sub-agents via output placeholders. | [agents.py](file:///Users/manohar/ADK_Basic/b1_sequential_agent/agents.py) |
| [**b2_parallel_agent**](file:///Users/manohar/ADK_Basic/b2_parallel_agent) | Fan-Out & Synthesize | `ParallelAgent` execution followed by a synthesis step in sequence. | [agents.py](file:///Users/manohar/ADK_Basic/b2_parallel_agent/agents.py) |
| [**b3_loop_agent**](file:///Users/manohar/ADK_Basic/b3_loop_agent) | Critique & Refine | `LoopAgent` iterative refinement cycle under check constraints, custom exit tools. | [agents.py](file:///Users/manohar/ADK_Basic/b3_loop_agent/agents.py) |
| [**b4_manual_sequential_flow**](file:///Users/manohar/ADK_Basic/b4_manual_sequential_flow) | Manual Pipeline | Python code-driven sequence orchestration, custom parsing, and manual session routing. | [agent.py](file:///Users/manohar/ADK_Basic/b4_manual_sequential_flow/agent.py), [main.py](file:///Users/manohar/ADK_Basic/b4_manual_sequential_flow/main.py) |
| [**c_custom_agent**](file:///Users/manohar/ADK_Basic/c_custom_agent) | Dynamic Orchestrator | Inherits `BaseAgent`, custom async generators, runtime loop decision gates in Python. | [agents.py](file:///Users/manohar/ADK_Basic/c_custom_agent/agents.py) |
| [**d_routing_agent**](file:///Users/manohar/ADK_Basic/d_routing_agent) | Master Request Router | Router evaluates prompt properties and routes to specialized workflows. | [agents.py](file:///Users/manohar/ADK_Basic/d_routing_agent/agents.py) |
| [**e_agent_as_tool**](file:///Users/manohar/ADK_Basic/e_agent_as_tool) | Hierarchical Architect | Wrapping agents using `AgentTool` class; dynamic invocation within instruction loops. | [agents.py](file:///Users/manohar/ADK_Basic/e_agent_as_tool/agents.py) |
| [**f_agent_with_memory**](file:///Users/manohar/ADK_Basic/f_agent_with_memory) | State-Based Memory | Custom memory tools manipulating `tool_context.state` (persisted sessions). | [agents.py](file:///Users/manohar/ADK_Basic/f_agent_with_memory/agents.py), [main.py](file:///Users/manohar/ADK_Basic/f_agent_with_memory/main.py) |
| [**g_agents_mcp**](file:///Users/manohar/ADK_Basic/g_agents_mcp) | MCP Tool Server Integration | Load and execute Model Context Protocol (MCP) toolsets via `ToolboxSyncClient`. | [trip_agent.py](file:///Users/manohar/ADK_Basic/g_agents_mcp/trip_agent.py), [main.py](file:///Users/manohar/ADK_Basic/g_agents_mcp/main.py) |
| [**mcp_tool_box**](file:///Users/manohar/ADK_Basic/mcp_tool_box) | MCP Toolbox Server | SQL declarations, tool mappings, and sqlite-sql statements. | [trip_tools.yaml](file:///Users/manohar/ADK_Basic/mcp_tool_box/trip_tools.yaml) |

---

## 🛠️ Setup & Installation

### Prerequisites
- **Python 3.8+** (Python 3.9+ recommended for the latest ADK features)
- **Google Cloud SDK** installed and configured
- **Google Cloud Project** with Vertex AI enabled

### Virtual Environment & Dependencies Setup
The repository includes automated environment setup scripts.

**On macOS/Linux:**
```bash
chmod +x setup_venv.sh
./setup_venv.sh
```

**On Windows:**
```cmd
setup_venv.bat
```

These scripts will:
1. Validate your Python version.
2. Initialize a Python virtual environment in `.adk_env`.
3. Install required libraries from `requirements.txt`.
4. Prompt you for your Google Cloud Project ID and authenticate via `gcloud`.
5. Create a `.env` file containing configuration variables:
   ```env
   GOOGLE_GENAI_USE_VERTEXAI=TRUE
   GOOGLE_CLOUD_PROJECT=your_project_id
   GOOGLE_CLOUD_LOCATION=us-central1
   ```

---

## 🏛️ Agent Design Patterns Deep Dive

### 1. Single Agent Pattern (`a_single_agent`)
A basic agent configuration that is given access to a search tool. The agent acts autonomously based on standard instructions and prompts.
- **Workflow**:
  ```mermaid
  graph LR
      User([User Query]) --> Agent[Single Agent]
      Agent -->|Tool Use| Search[Google Search]
      Search -->|Results| Agent
      Agent --> Final([Final Output])
  ```
- **Example Usage**:
  ```python
  from google.adk.agents import Agent
  from google.adk.tools import google_search

  planner_agent = Agent(
      name="planner_agent",
      model="gemini-2.5-flash",
      instruction="Plan a day trip...",
      tools=[google_search]
  )
  ```

---

### 2. Sequential Agent Pipeline (`b1_sequential_agent`)
Chains multiple agents linearly. Output variables from prior steps are saved to session state and injected into subsequent agent prompts using `{state_key}` placeholders.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Seq[SequentialAgent: find_and_navigate_agent]
      Seq --> Agent1[Foodie Agent]
      Agent1 -->|Output: destination| State[Shared Session State]
      State -->|Input: {destination}| Agent2[Transportation Agent]
      Agent2 --> Final([Final Directions])
  ```
- **Key Concepts**:
  - `SequentialAgent`: Orchestrates sub-agents in a strict linear order.
  - `output_key`: Directs the framework to capture final output under a specific key in the session state.

---

### 3. Parallel Agent Workflow (`b2_parallel_agent`)
Fans out queries to multiple specialized sub-agents running concurrently, then forwards all outputs to a synthesis agent to compile the final summary.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Seq[SequentialAgent: parallel_planner_agent]
      Seq --> Par[ParallelAgent: parallel_research_agent]
      subgraph Parallel Search
          Par --> A1[Museum Finder Agent]
          Par --> A2[Concert Finder Agent]
          Par --> A3[Restaurant Finder Agent]
      end
      A1 -->|museum_result| State[Shared Session State]
      A2 -->|concert_result| State
      A3 -->|restaurant_result| State
      State --> Synth[Synthesis Agent]
      Synth --> Final([Synthesized Bulleted List])
  ```
- **Key Concepts**:
  - `ParallelAgent`: Executes sub-agents concurrently.
  - State values like `{museum_result}` and `{concert_result}` are automatically bound into the synthesis agent's instruction variables.

---

### 4. Critique & Refine Loop (`b3_loop_agent`)
Executes an iterative improvement cycle. A critic agent inspects constraints, and a refiner agent corrects the plan. This continues until the exit agent calls the loop termination tool.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Seq[SequentialAgent: iterative_planner_agent]
      Seq --> Plan[Planner Agent]
      Plan -->|current_plan| Loop[LoopAgent: refinement_loop]
      subgraph Loop Iterations (Max 3)
          Loop --> Critic[Critic Agent]
          Critic -->|Travel time > 45m?| Critique{Critique}
          Critique -->|Yes: Provide feedback| Refine[Refiner Agent]
          Refine -->|Updated plan| Critic
          Critique -->|No: Completion phrase| ExitAgent[Exit Agent]
          ExitAgent -->|Calls exit_loop tool| Exit[Exit Loop]
      end
      Exit --> Final([Feasible Trip Plan])
  ```
- **Key Concepts**:
  - `LoopAgent`: Runs sub-agents in a repeating loop.
  - `tool_context.actions.escalate = True` is used in custom tools to break the execution loop and bubble up the final state.

---

### 5. Manual Control Flow Workflow (`b4_manual_sequential_flow`)
Shows how to achieve complex multi-agent orchestration programmatically using standard async Python syntax. This provides total flexibility for custom regex extractions and session transitions.
- **Key Concepts**:
  - Useful for debugging or migrating existing Python scripts without using ADK's specialized Agent containers.
  - Showcases custom route analysis via a simple `router_agent` and code-guided execution transitions.

---

### 6. Custom Agent Dynamic Orchestration (`c_custom_agent`)
Inherits from `BaseAgent` and implements the custom runner logic inside the `_run_async_impl` method. This allows wrapping raw Python check gates, custom try/except handlers, budget deductions, and dynamic branching conditions directly in Python.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Coordinator[BudgetAwarePlannerAgent]
      Coordinator --> Parse[BudgetParserAgent]
      Parse -->|total_budget| Decision1{Valid Budget?}
      Decision1 -->|No| Fail[FailureAgent: End conversation]
      Decision1 -->|Yes| PlanAct[ActivityFinderAgent]
      PlanAct -->|found_activity| CostAct[CostEstimatorAgent]
      CostAct -->|estimated_cost| Decision2{Under Budget?}
      Decision2 -->|Yes| AddAct[Add Activity & Update total]
      Decision2 -->|No| SkipAct[Skip Activity]
      AddAct & SkipAct --> PlanRest[RestaurantFinderAgent]
      PlanRest -->|found_restaurant| Decision3{Meal Under Budget?}
      Decision3 -->|Yes| AddRest[Add Restaurant & Update total]
      Decision3 -->|No| SkipRest[Skip Restaurant]
      AddRest & SkipRest --> Synth[SummaryAgent]
      Synth --> Final([Final Itinerary Summary])
  ```
- **Key Concepts**:
  - Overriding `async def _run_async_impl(self, ctx: InvocationContext) -> AsyncGenerator[Event, None]`.
  - Full control over events yielded back to the client interface.

---

### 7. Request Router Agent (`d_routing_agent`)
A master routing agent analyzes a request and delegates execution to one of several specialized workflows based on semantic keywords (e.g. money -> budget planner, loop constraints -> iterative planner, multiple items -> parallel planner).
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Router[Router Agent]
      Router -->|Analyze constraints & budget| Dec{Choose Route}
      Dec -->|Has Budget| A[Budget Planner Agent]
      Dec -->|Has Travel Time Constraint| B[Iterative Planner Agent]
      Dec -->|Multiple Diverse Requests| C[Parallel Planner Agent]
      Dec -->|Simple Day Trip| D[Day Trip Workflow]
      Dec -->|Simple Weekend Events| E[Weekend Guide Workflow]
      Dec -->|Find & Navigate| F[Find and Navigate Agent]
      A & B & C & D & E & F --> Final([Execute and Return Response])
  ```

---

### 8. Hierarchical Agent-as-a-Tool Pattern (`e_agent_as_tool`)
Allows wrapping a sub-agent as a tool via `AgentTool`. The main orchestrator agent determines when, how, and with what arguments to call the sub-agent.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Arch[TripArchitectAgent]
      subgraph Agent Tools
          Arch -->|Call as Tool| Scout[LocationScoutAgent]
          Arch -->|Call as Tool| Val[LogisticsValidatorAgent]
      end
      Scout -->|Returns location name| Arch
      Val -->|Returns travel time/hours| Arch
      Arch -->|Auto self-corrects if logistics fail| Arch
      Arch --> Final([Final Logistically Sound Itinerary])
  ```
- **Key Concepts**:
  - `AgentTool(agent=sub_agent)`: Turns any ADK Agent instance into a call-compatible function within the model's tool schema.

---

### 9. State-Based Session Memory (`f_agent_with_memory`)
Saves, merges, and recalls user preferences using persistent session storage.
- **Workflow**:
  ```mermaid
  graph TD
      User([User Input]) --> Coord[MemoryCoordinatorAgent]
      Coord -->|1. Call Tool| Recall[recall_user_preferences]
      Recall -->|Read State| State[(Session State)]
      State -->|Preferences| Coord
      Coord -->|2. Enrich & Call| Planner[PlannerToolAgent]
      Planner -->|Generate Plan| Coord
      Coord -->|3. Present & Ask for feedback| UserFeedback([User Feedback / New Prefs])
      UserFeedback -->|4. If new pref, Call Tool| Save[save_user_preferences]
      Save -->|Write State| State
  ```
- **Key Concepts**:
  - Reading from and writing to `tool_context.state` (which maps back to the persistent database schema).

---

### 10. Model Context Protocol (MCP) Integration (`g_agents_mcp` / `mcp_tool_box`)
Loads and calls structured toolsets exposed via an external Model Context Protocol (MCP) server. Tools are defined in a database declaration YAML and read into the agent runtime.
- **Workflow**:
  ```mermaid
  graph LR
      Agent[Trip Planner Agent] -->|Request Tools| MCP[MCP Toolbox Client]
      MCP -->|Expose SQLite SQL| Server[Toolbox Server on :7001]
      Server --> DB[(destinations.db)]
      Agent -->|Invoke Tool| MCP
      MCP -->|Run SQL Query| Server
      Server -->|Results| Agent
  ```
- **Key Concepts**:
  - `ToolboxSyncClient("http://127.0.0.1:7001")`: Connection interface to the MCP Toolbox.
  - `toolbox.load_toolset("trip-planner-tools")`: Fetching query declarations dynamically at runtime.

---

## 🏃 Running the Code

### 1. CLI Execution
Ensure your virtual environment is active:
```bash
source .adk_env/bin/activate
```

Run manual flow scripts, memory, or MCP agent entries directly:
```bash
# Run manual sequential flow orchestration
python b4_manual_sequential_flow/main.py

# Run persistent memory coordinator cli
python f_agent_with_memory/main.py

# Setup SQLite DB for MCP
python setup_trip_database.py
# Start the MCP Client entrypoint (requires MCP toolbox server running on port 7001)
python g_agents_mcp/main.py
```

### 2. Web Interface (ADK Web UI)
Start the ADK Web UI locally on http://localhost:8080 with persistent session tracking using the provided script:
```bash
./run.sh
```

This starts the application using:
```bash
adk web --session_service_uri="sqlite:///$HOME/.adk/sessions/adk_web_sessions.db" --host=127.0.0.1 --port=8080 .
```
Any changes you make inside the Web UI (such as saved memory variables) will persist across browser sessions.
