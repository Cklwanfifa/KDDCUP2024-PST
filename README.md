# KDDCUP2024-PST
The top3 solution of KDD CUP 2024 PST challenge


## 本目录为队伍”英国大力士”在KDD CUP 2024 PST 竞赛复赛第三名的解决方案介绍。

本方案主要参考了"More Agents Is All You Need" (https://arxiv.org/abs/2402.05120) 的思路，利用大量不同的大语言模型（包括GPT4-turbo, GPT4o, Gemini pro 1.5, Claude 3 Opus）并设计多样性的prompt, 对结果的预测做整合，得到最终的答案。
其最大的特点是，不需要本地训练深度学习模型，因此可能是前排中唯一不需要显存的解决方案。缺点是为了输出答案可能需要耗费～500美金左右的Token费用。如果包括竞赛中途的一些无效尝试，总耗费金额可能接近1000美金。


