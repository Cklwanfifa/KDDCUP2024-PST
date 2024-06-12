# KDDCUP2024-PST
The top3 solution of KDD CUP 2024 PST challenge


## 本目录为队伍”英国大力士”在KDD CUP 2024 PST 竞赛复赛第三名的解决方案介绍。

本方案主要参考了"More Agents Is All You Need" (https://arxiv.org/abs/2402.05120) 的思路，利用大量不同的大语言模型（包括GPT4-turbo, GPT4o, Gemini pro 1.5, Claude 3 Opus）并设计多样性的prompt, 对结果的预测做整合，得到最终的答案。
其最大的特点是，不需要本地训练深度学习模型，因此可能是前排中唯一不需要填写显存与模型参数量的解决方案。缺点是1. 为了输出答案需要耗费价值约500美金左右的Token费用。如果包括竞赛过程的一些无效尝试，总耗费金额可能更多。2.无法利用训练集标签提高让LLM能更加理解关键引用的准确定义。

具体的做法非常简单，我们将每一篇论文的正文部分处理成只剩下文本和引用序号的部分。如"aaa bbb ccc[b1], ddd eee fff[b2] ..."。然后直接询问闭源模型，哪些引用可能是关键引用，并直接返回序号（b1, b2...）。

## 主要步骤

### 一、运行feature_engineering.ipynb。
这个脚本的功能是
1. 处理paper，包括异常值填充和特征抽取，并将一些处理好的特征保存在processed_data_0601.pickle中。
2. 导出prompt。我们设计如下几种不同的prompt:

   2.1 给出关键引用的定义，让LLM直接给出关键引用的结果以及对应的置信度；

   2.2 聚焦于"inspiration"这个典型的关键词，让LLM找出属于"direct inspiration"，"indirect inspiration", "other inspiration"的引用。

   2.3 和2.1类似，但是会给出每个引用文章的标题。

   2.4 和2.1类似，但是会先让GPT4自己优化一遍prompt.

   2.5 对于数据集中包含“note”字段的文章，让LLM基于note的描述内容去寻找关键引用。

最后将不同Prompt的结果保存。

### 二、多次调用闭源API获取LLM agent给出的答案。

#### Notes：1. 由于是自费参赛且经费有限，对于部分的LLM的结果只刷了test数据集的部分。2.参赛过程比较仓促，导致这部分文件名的命名会比较随意。3. 我的Gemini Pro的API KEY很容易触发限额，因此代码实现上每调用一次需等待30s，并分成50个一组运行分批保存避免遇到错误。4. 原始提交数据依赖的LLM agent给出的答案已经附在目录中。

##### OPUS部分
1. 运行20240601_opus_1.ipynb，填入自己的API_KEY，结果保存到opus_res_parse_test.json;

##### GEMINI部分
1. 运行Gemini_pro_test_1.ipynb，填入自己的API_KEY，结果保存在gemini_test_result_all.pkl;
  
2. 运行Gemini_pro_test_2.ipynb，填入自己的API_KEY，结果保存在gemini_test_result_all_round2.pkl;
 
3. 运行Gemini_pro_test_3.ipynb，填入自己的API_KEY，结果保存在gemini_test_result_all_round3.pkl;
 
4. 运行Gemini_pro_test_4.ipynb，填入自己的API_KEY，结果保存在gemini_test_result_all_round4.pkl;

#### GPT4部分
1. 运行get_GPT_res_level.ipynb, 填入自己的API_KEY，结果保存到gpt4_res_parse_train_level.json, gpt4_res_parse_valid_level.json, gpt4_res_parse_test_level.json.

2. 运行get_GPT_res_short.ipynb，填入自己的API_KEY，结果保存到gpt4_res_parse_train.json, gpt4_res_parse_valid.json, gpt4_res_parse_test_short.json.

3. 运行get_gpt4_only_test_result.ipynb，填入自己的API_KEY，结果保存到gpt4_res_parse_test_only.json, gpt4_res_parse_test_only_V2.json, GPT4_res_test_note.json, GPT4_res_train_V2_save.json, GPT4_res_test_V3_save.json.

### 三、整合LLM模型结果
由于很多情况下LLM输出的结果依然以文本形式存在，为了获得json格式的答案，我们还需要去解析并整合不同LLM模型的结果。

#### Notes：1.原始提交数据依赖的LLM agent给出的答案已经附在目录中。

运行parse_gpt_result.ipynb，在这个脚本中我们会

1. 解析gpt4_res_parse_train_level.json，gpt4_res_parse_valid_level.json，gpt4_res_parse_test_level.json的结果，并保存到gpt4_res_parse_train_level_parsed.json, gpt4_res_parse_valid_level_parsed.json, gpt4_res_parse_test_level_parsed.json.

2. 解析gpt4_res_parse_train.json, gpt4_res_parse_valid.json, gpt4_res_parse_test_short.json的结果，并保存到gpt4_res_parse_train_short_parsed.json, gpt4_res_parse_valid_short_parsed.json, gpt4_res_parse_test_short_parsed.json.

3. 解析opus_res_parse_test.json的结果，这里需要调用GPT3.5。结果保存到opus_res_parse_json_test.json

4. 解析gpt4_res_parse_test_only.json的结果，结果保存到gpt4_turbo_res_parse_json_test.json

5. 解析gpt4_res_parse_test_only_V2.json的结果，这里需要调用GPT3.5。结果保存到gpt4_turbo_res_parse_json_test_v2.json

6. 解析GPT4_res_test_note.json的结果，结果保存到GPT4_res_test_note_parse.json

7. 解析gemini_test_result_all.pkl, gemini_test_result_all_round2.pkl, gemini_test_result_all_round3.pkl, gemini_test_result_all_round4.pkl的结果，并把结果保存到gemini_res_parse_json_test.json, gemini_res_parse_json_test_round2.json, gemini_res_parse_json_test_round3.json, gemini_res_parse_json_test_round4.json.

8. 解析GPT4_res_test_V3_save.json，结果保存在GPT4_res_test_V3_parse.json。

### 输出最后的结果

运行feature_merge.ipynb

1. 导入之前生成的文本特征和LLM预测的结果；
   
2. 在训练集训练Lightgbm和Catboost模型，输入为文本特征和部分在训练集有训练数据的LLM预测的结果，并在测试集上预测分数， 得到原始预测分数pred。
   
3. 让其他仅在测试集进行预测的LLM agent结果对结果进行修正”:
   
   3.1 加分步骤：将LLM agent进行分组(主要基于基座模型和prompt类型)，每一个组内对于结果进行投票，并加分到对应的pred分数上。
   
   3.2 扣分步骤：将所有LLM agent的结果放置在一起进行排序，对于每一个reference，如果其某个特定分位数的LLM的置信分数小于某个threshold，则进行扣分。
   
这样做的思路是，在我们之前的一些实验中，发现不同基座的LLM往往是站在在不同的角度对reference进行打分，因此可以基于各自的结果对pred进行投票加分。但同时，如果存在仅仅部分LLM agent给出了比较置信的分数的情况，可能说明这部分的LLM agent存在误判，这个时候需要参考集体的投票情况，进行扣分操作。这样的步骤可以被总结成”分组性加分，一致性扣分“.

4. 导出最后的结果在sample_submission_0607_1.json。



