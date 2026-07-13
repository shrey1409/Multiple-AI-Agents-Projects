# Important AI Agents — Curated from 500-AI-Agents-Projects

Curated for: multi-agent systems, RAG/document agents, domain apps, dev tools (advanced level).
Source: [github.com/shrey1409/500-AI-Agents-Projects](https://github.com/shrey1409/500-AI-Agents-Projects)

---

## 1. Multi-Agent Orchestration (highest value for you)

These are the core architecture patterns every advanced multi-agent project is built on.

| Agent | Framework | Why it matters | Link |
|---|---|---|---|
| Multi-Agent Supervisor | LangGraph | The canonical supervisor→worker pattern; base for almost any agent team | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/multi_agent/agent_supervisor.ipynb) |
| Hierarchical Agent Teams | LangGraph | Supervisor-of-supervisors; scales to large agent fleets | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/multi_agent/hierarchical_agent_teams.ipynb) |
| Multi-Agent Collaboration | LangGraph | Peer-to-peer specialist agents sharing state on complex tasks | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/multi_agent/multi-agent-collaboration.ipynb) |
| Plan-and-Execute Agent | LangGraph | Planner generates multi-step plan, executor runs it; core agentic pattern | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/plan-and-execute/plan-and-execute.ipynb) |
| Reflection Agent | LangGraph | Agent critiques and revises its own output — big quality gains | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/reflection/reflection.ipynb) |
| Reflexion Agent | LangGraph | Iterative self-improvement with episodic memory | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/reflexion/reflexion.ipynb) |
| Group Chat w/ Custom Speaker Selection | AutoGen | Fine-grained control over which agent speaks when | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_groupchat_customized) |
| Task Solving via Graph Transition Paths (FSM) | AutoGen | Constrains agent handoffs with a finite state machine — production-grade control | [Notebook](https://microsoft.github.io/autogen/docs/notebooks/agentchat_groupchat_finite_state_machine) |
| Nested Chats for Complex Tasks | AutoGen | Hierarchical problem decomposition via nested conversations | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_nestedchat) |
| SocietyOfMindAgent | AutoGen | Inner-monologue simulation; interesting research-grade pattern | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_society_of_mind) |
| AgentBuilder (auto-build agent systems) | AutoGen | Meta: an agent that builds multi-agent systems | [Notebook](https://github.com/microsoft/autogen/blob/0.2/notebook/autobuild_basic.ipynb) |
| CrewAI + LangGraph Integration | CrewAI | Combines role-based crews with stateful graphs | [Repo](https://github.com/crewAIInc/crewAI-examples/tree/main/integrations/CrewAI-LangGraph) |
| Citadel | Claude Code | Orchestrates Claude Code agent fleets with lifecycle hooks, skills, campaign management | [Repo](https://github.com/SethGammon/Citadel) |

## 2. Advanced RAG / Document Agents

The four LangGraph RAG variants below are the state of the art in agentic retrieval — knowing all four is a differentiator.

| Agent | Framework | Why it matters | Link |
|---|---|---|---|
| Agentic RAG | LangGraph | Agent picks retrieval strategy before answering | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_agentic_rag.ipynb) |
| Adaptive RAG | LangGraph | Routes queries by complexity (no-retrieval / single / multi-step) | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_adaptive_rag.ipynb) |
| Corrective RAG (CRAG) | LangGraph | Grades retrieved docs, falls back to web search when they're bad | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_crag.ipynb) |
| Self-RAG | LangGraph | Reflects on its own answer and re-retrieves if unsupported | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_self_rag.ipynb) |
| Adaptive/Self-RAG (local models) | LangGraph | Same patterns fully offline with local LLMs | [Adaptive](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_adaptive_rag_local.ipynb) · [Self](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/rag/langgraph_self_rag_local.ipynb) |
| RAG Group Chat | AutoGen | Multi-agent collaboration with shared retrieval | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_groupchat_RAG) |
| DeepKnowledge | Agno | Iterative knowledge-base search with deep reasoning | [Code](https://github.com/agno-agi/agno/blob/main/cookbook/examples/agents/deep_knowledge.py) |
| Legal Document Analysis Agent | Agno | Vector-embedding analysis of legal PDFs — good domain RAG example | [Code](https://github.com/agno-agi/agno/blob/main/cookbook/examples/agents/legal_consultant.py) |
| Legal Document Review Assistant | — | Clause extraction and review automation | [Repo](https://github.com/firica/legalai) |

## 3. Dev & Productivity Agents

| Agent | Framework | Why it matters | Link |
|---|---|---|---|
| Code Assistant (self-correcting) | LangGraph | Generates code, runs checks, iteratively fixes errors | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/code_assistant/langgraph_code_assistant.ipynb) |
| Auto Code Gen + Execution + Debugging | AutoGen | The classic self-healing coding loop | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_auto_feedback_from_code_execution) |
| Code Gen + Q&A with Retrieval (RetrieveChat) | AutoGen | RAG over a codebase; Qdrant variant available | [Notebook](https://microsoft.github.io/autogen/0.2/docs/notebooks/agentchat_RetrieveChat) |
| SQL Agent (NL → SQL over DBs) | LangGraph | Natural-language database querying with schema awareness | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/sql-agent.ipynb) |
| NL to SQL Query | AutoGen | Spider-benchmark NL→SQL approach | [Notebook](https://github.com/microsoft/autogen/blob/0.2/notebook/agentchat_sql_spider.ipynb) |
| Readme Generator Agent | Agno | Auto-generates quality READMEs for repos | [Code](https://github.com/agno-agi/agno/blob/main/cookbook/examples/agents/readme_generator.py) |
| Email Auto Responder Flow | CrewAI | Practical automation flow pattern | [Repo](https://github.com/crewAIInc/crewAI-examples/tree/main/flows/email_auto_responder_flow) |
| Meeting Assistant + Prep-for-Meeting | CrewAI | Scheduling, agenda prep, meeting materials | [Flow](https://github.com/crewAIInc/crewAI-examples/tree/main/flows/meeting_assistant_flow) · [Crew](https://github.com/crewAIInc/crewAI-examples/tree/main/crews/prep-for-a-meeting) |
| Chatbot Simulation Evaluation | LangGraph | Simulated-user evals for your agents — critical for serious projects | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/chatbot-simulation-evaluation/agent-simulation-evaluation.ipynb) |
| AgentEval + AgentOps | AutoGen | Evaluation and observability for agent systems | [AgentEval](https://github.com/microsoft/autogen/blob/0.2/notebook/agenteval_cq_math.ipynb) · [AgentOps](https://github.com/microsoft/autogen/blob/0.2/notebook/agentchat_agentops.ipynb) |

## 4. Domain Apps (portfolio-worthy)

| Agent | Domain | Why it matters | Link |
|---|---|---|---|
| AI Health Assistant (Medical Diagnostics) | Healthcare | Multi-agent medical diagnosis — strong multi-agent + domain combo | [Repo](https://github.com/ahmadvh/AI-Agents-for-Medical-Diagnostics) |
| HIA — Health Insights Agent | Healthcare | Medical report analysis and insights | [Repo](https://github.com/harshhh28/hia) |
| MediSuite-AI-Agent | Health Insurance | End-to-end hospital/insurance claim workflow automation | [Repo](https://github.com/ahmedmansour5/MediSuite-Ai-Agent) |
| StockAgent — Automated Trading | Finance | Multi-agent stock trading simulation with market analysis | [Repo](https://github.com/MingyuJ666/Stockagent) |
| Stock Analysis Crew | Finance | Role-based financial analysis team | [Repo](https://github.com/crewAIInc/crewAI-examples/tree/main/crews/stock_analysis) |
| Financial Reasoning Agent | Finance | Claude-based stock analysis with Yahoo Finance data | [Code](https://github.com/agno-agi/agno/blob/main/cookbook/examples/agents/reasoning_finance_agent.py) |
| Agent Wallet SDK | Finance/Web3 | Non-custodial wallets for AI agents with spend limits — emerging space | [Repo](https://github.com/up2itnow0822/agent-wallet-sdk) |
| Customer Support Agent | Customer Service | Production-style LangGraph support agent (flight booking, policies, tools) | [Notebook](https://github.com/langchain-ai/langgraph/blob/main/docs/docs/tutorials/customer-support/customer-support.ipynb) |
| Real-Time Threat Detection | Cybersecurity | LLM agents for security operations | [Repo](https://github.com/NVISOsecurity/cyber-security-llm-agents) |
| Decepticon — Vibe Hacking Agent | Cybersecurity | Autonomous multi-agent red-team testing | [Repo](https://github.com/PurpleAILAB/Decepticon) |
| OptiGuide — Logistics Optimization | Supply Chain | Microsoft's LLM + optimization-solver hybrid; also has an AutoGen nested-chat version | [Repo](https://github.com/microsoft/OptiGuide) |
| RecAI — Product Recommendation | Retail | Microsoft's recommender-agent framework | [Repo](https://github.com/microsoft/RecAI) |
| AI Travel Agent | Travel | Itinerary planning; compare with CrewAI [Trip Planner](https://github.com/crewAIInc/crewAI-examples/tree/main/crews/trip_planner) | [Repo](https://github.com/nirbar1985/ai-travel-agent) |
| EduGPT — Virtual AI Tutor | Education | Personalized tutoring flows | [Repo](https://github.com/hqanhh/EduGPT) |
| Jobber — Recruitment Agent | HR | Browser-automation agent that applies to jobs | [Repo](https://github.com/sentient-engineering/jobber) |

## 5. Future Build Ideas (combine the patterns above)

Ranked by portfolio impact for an advanced builder:

1. **Multi-agent research analyst** — Supervisor (LangGraph) + Corrective RAG + Financial Reasoning Agent tools + Reflection for report QA. One project that demos orchestration, RAG, and a domain.
2. **Self-healing data pipeline agent** — Plan-and-Execute + SQL Agent + AutoGen code-execution loop: agent writes queries, validates results, fixes its own errors.
3. **Agentic red-team/blue-team sandbox** — Decepticon-style attacker crew vs. Threat Detection defender crew with an FSM referee (AutoGen graph transitions). Defensive/CTF-lab scope only.
4. **Claims-processing pipeline** — MediSuite-style workflow rebuilt as hierarchical LangGraph teams: intake → document RAG → validation → human-in-the-loop approval.
5. **Codebase onboarding agent** — RetrieveChat over a repo + Readme Generator + Code Assistant, wrapped in one supervisor with an eval harness (Chatbot Simulation Evaluation).
6. **Agent evaluation dashboard** — AgentEval + AgentOps + simulated users; few people build eval tooling, so it stands out.

**Pattern to reuse everywhere:** Supervisor → specialist workers → reflection/eval loop. Every top project in the repo is a variation of it.
