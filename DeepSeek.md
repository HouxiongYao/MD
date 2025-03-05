# DeepSeek

## 问题背景

大模型（gpt-o1）通过延长思维链（Chain-of-Thought）的推理步骤在生成答案时让模型进入更深入的逻辑推导，但是在test time scaling上不够高效。

提出DeepSeek-R1系列。DeepSeek-R1 zero不使用监督数据，大规模运用强化学习。DeepSeek-R1系列在此基础上解决了zero的poor readability, and language mixing等问题。使用了两个RL阶段和两个SFT阶段。

## Approach





