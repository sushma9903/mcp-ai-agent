# 🤖 MCP AI Agent with LangGraph

An intelligent AI agent that connects to Model Context Protocol (MCP) servers, dynamically discovers tools, and maintains conversation memory using LangGraph.

---

## 📋 Overview

This project implements an AI agent that:
- Connects to MCP servers via STDIO transport
- Dynamically discovers and invokes tools based on user intent
- Maintains full conversation history and context
- Uses LangGraph for robust state management
- Supports weather queries, stock prices, and web search

---

## 🏗️ Architecture

```
┌───────────────────────────────────────────────────────────┐
│                          User                             │
│              (Natural Language Queries)                   │
└─────────────────────────────┬─────────────────────────────┘
                              │
                              ▼
┌───────────────────────────────────────────────────────────┐
│                  LangGraph AI Agent                       │
│                       (agent.py)                          │
│                                                           │
│   ┌───────────────────────────────────────────────────┐   │
│   │                  Agent Node (LLM)                 │   │
│   │                                                   │   │
│   │ • Interprets user intent                          │   │
│   │ • Uses full conversation history                  │   │
│   │ • Decides whether a tool call is required         │   │
│   └───────────────────────┬───────────────────────────┘   │
│                           │                               │
│   ┌───────────────────────▼───────────────────────────┐   │
│   │                 Tool Node (Executors)             │   │
│   │                                                   │   │
│   │ • Executes selected tools                         │   │
│   │ • Handles tool inputs and outputs                 │   │
│   │ • Returns results back to the agent               │   │
│   └───────────────────────┬───────────────────────────┘   │
└───────────────────────────┼───────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────┐
│                    MCP Client (STDIO)                     │
│                                                           │
│ • Spawns MCP server as a subprocess                       │
│ • Discovers available tools dynamically                   │
│ • Sends and receives MCP protocol messages                │
└───────────────────────────┬───────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────┐
│                      MCP Server                           │
│                    (server/main.py)                       │
│                                                           │
│ • get_weather                                             │
│ • get_stock_price                                         │
│ • web_search                                              │
│                                                           │
│ Exposes tools via MCP with JSON schemas                   │
│ Contains no agent or decision logic                       │
└───────────────────────────────────────────────────────────┘

```

---

## 🚀 Quick Start

### **Prerequisites**

- Python 3.9+
- Groq API key
- MCP server (included in `server/main.py`)

### **Installation**

1. **Clone the repository**
   ```bash
   cd task2-mcp+agent
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set up environment variables**
   
   Create a `.env` file in the root directory:
   ```env
   GROQ_API_KEY=your_groq_api_key_here
   ```

4. **Run the agent**
   ```bash
   python agent/agent.py
   ```

---

## 📦 Dependencies

```txt
langchain-groq>=0.2.0
langchain>=0.3.0
langgraph>=0.2.0
python-dotenv>=1.0.0
mcp>=1.0.0
pydantic>=2.0.0
```

Install all at once:
```bash
pip install langchain-groq langchain langgraph python-dotenv mcp pydantic
```

---

## 🎯 Usage Examples

### **Basic Queries**

```
You: What is the weather in Paris?
Agent: The current weather in Paris is a clear sky with a temperature 
of 3.87°C, humidity of 73%, and wind speed of 4.12 m/s.

You: What is the stock price of TSLA?
Agent: The current stock price of TSLA is $485.4.

You: Search the web for latest AI news
Agent: Based on the search results, here are the latest AI news:
* Google has announced its first AI hub in India...
```

### **Context-Aware Conversations**

```
You: What is the weather in London?
Agent: The current weather in London is 6.04°C...

You: What about Tokyo?
Agent: The current weather in Tokyo is 2.97°C...

You: What about the previous city?
Agent: London.
```

### **Memory Recall**

```
You: What questions have I asked you?
Agent: You have asked me the following questions:
1. What is the weather in Paris?
2. What about bangalore?
3. What is the stock price of TSLA?
...
```

---

## 🔧 Technical Details

### **Component Breakdown**

#### **1. MCPClient Class**
- Manages STDIO connection to MCP server
- Spawns server process as subprocess
- Handles bidirectional communication
- Discovers available tools via MCP protocol

#### **2. Tool Conversion**
- Converts MCP tool schemas to LangChain StructuredTools
- Maps JSON schema types to Python types (string→str, integer→int)
- Creates Pydantic models for input validation
- Wraps tool execution in async functions

#### **3. LangGraph Agent**
- **StateGraph**: Manages conversation flow
- **Agent Node**: LLM decision-making with full conversation history
- **Tool Node**: Executes tool calls in parallel
- **Conditional Routing**: Decides whether to use tools or respond

#### **4. Memory System**
- Maintains full conversation history as message list
- Passes complete context to LLM on every turn
- Supports contextual understanding and recall
- No external database required (in-memory)

### **Message Flow**

```
User Input
    ↓
[Add to conversation_history]
    ↓
[Invoke LangGraph agent with full history]
    ↓
Agent Node: LLM analyzes history + new question
    ↓
    ├─→ Needs tool? → Tool Node → Execute → Back to Agent
    └─→ Can answer? → Generate response → Return
    ↓
[Add response to conversation_history]
    ↓
Display to User
```

---

## 🛠️ Customization

### **Adding New Tools**

1. Add tool to your MCP server (`server/main.py`)
2. Agent will automatically discover it on startup
3. No changes needed to agent code!

### **Changing LLM Model**

```python
llm = ChatGroq(
    groq_api_key=GROQ_API_KEY,
    model_name="llama-3.1-70b-versatile",  # Change model
    temperature=0
)
```

### **Adjusting System Prompt**

Edit the `system_message` in `run_agent()` function:
```python
system_message = SystemMessage(content="""
Your custom instructions here...
""")
```

---

## 🧪 Testing

### **Test Suite**

Run these prompts to verify functionality:

**Tool Invocation:**
```
What is the weather in Paris?
What is the stock price of AAPL?
Search the web for Python 3.12 features
```

**Memory & Context:**
```
What is the weather in London?
What about Tokyo?
What was the temperature in the previous city?
```

**Memory Recall:**
```
What questions have I asked?
What did we discuss earlier?
```

**Direct Answers:**
```
What is 5 + 7?
Tell me a joke
```

### **Expected Behavior**

- ✅ All tools execute successfully
- ✅ Agent maintains context across turns
- ✅ Can recall previous conversation
- ✅ Answers simple questions without tools
- ✅ No infinite loops or errors

---

## 🐛 Troubleshooting

### **Issue: "GROQ_API_KEY not found"**
**Solution:** Create `.env` file with your API key

### **Issue: "MCP session is not initialized"**
**Solution:** Ensure `server/main.py` exists and is executable

### **Issue: Tool schema validation errors**
**Solution:** Check that MCP server returns correct JSON schema types

### **Issue: Agent doesn't remember previous messages**
**Solution:** Verify `conversation_history` is being appended correctly

---

## 📚 Key Concepts

### **MCP (Model Context Protocol)**
- Standard protocol for connecting AI models to external tools
- Uses JSON-RPC over STDIO/HTTP for communication
- Servers expose tools with JSON schemas

### **LangGraph**
- Framework for building stateful, multi-step AI applications
- Uses directed graphs to control agent behavior
- Prevents common issues like infinite loops

### **ReAct Pattern**
- Reason + Act: LLM thinks, then uses tools, then responds
- LangGraph implements this with function calling (not text parsing)

### **Conversation Memory**
- Full history stored as list of messages
- Each turn: System + History + New Question
- Enables context understanding and recall

---

## 📖 References

- [LangGraph Documentation](https://docs.langchain.com/oss/python/langgraph/overview)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [LangChain Tools](https://python.langchain.com/docs/modules/tools/)

---

## 📄 License

This project is provided for educational purposes.

---

## 👤 Author

Built as part of MCP integration learning project.

---

## 🙏 Acknowledgments

- Anthropic for MCP specification
- LangChain team for LangGraph framework
- Groq for LLM API access
