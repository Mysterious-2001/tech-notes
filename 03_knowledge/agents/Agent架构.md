agent架构是整个应用架构的一部分，可以看成是关于如何组织agent完成任务的模式。
常见架构有：
1. 普通的Single-shot / Tool-augmented LLM
2. ReAct架构，reason+act，详见[[03_agents/ReAct|ReAct]]
3. Plan-and-execute架构，先规划再行动。Plan-execute-check-replan
4. Multi-agents,跟planner和executer这种架构的区别是有更多分工的agents。

在工业界其实很多agents系统本质是由workflow