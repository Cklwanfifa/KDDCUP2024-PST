# KDDCUP2024-PST
The top3 solution of KDD CUP 2024 PST challenge


## 本目录为队伍”英国大力士”在KDD CUP 2024 PST 竞赛复赛第三名的解决方案介绍。

本方案主要参考了"More Agents Is All You Need" (https://arxiv.org/abs/2402.05120) 的思路，利用大量不同的大语言模型（包括GPT4-turbo, GPT4o, Gemini pro 1.5, Claude 3 Opus）并设计多样性的prompt, 对结果的预测做整合，得到最终的答案。
其最大的特点是，不需要本地训练深度学习模型，因此可能是前排中唯一不需要显存的解决方案。缺点是1. 为了输出答案需要耗费价值约500美金左右的Token费用。如果包括竞赛过程的一些无效尝试，总耗费金额可能接近1000美金。2.无法利用训练集标签提高让LLM能更加理解关键引用的准确定义。

具体的做法非常简单，我们将每一篇论文的正文部分处理成只剩下文本和引用序号的部分。如"aaa bbb ccc[b1], ddd eee fff[b2] ..."。然后直接询问闭源模型，哪些引用可能是关键引用，并直接返回序号（b1, b2...）。

## 主要步骤

### 一、运行feature_engineering.ipynb。
这个脚本的功能是
1. 处理paper，包括异常值填充和特征抽取，并将一些处理好的特征保存在processed_data_0601.pickle中。
2. 导出Prompt。我们
