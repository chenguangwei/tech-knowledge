# NLP应用

## 1. NLP简介

NLP是自然语言处理（Natural Language Processing）的简称，它是人工智能的一个重要分支，涉及计算机和人类语言之间的交互。NLP的主要任务有两大类：**自然语言理解（NLU）和自然语言生成（NLG）**。**NLU是指让计算机理解人类语言的含义和意图，NLG是指让计算机用人类语言表达信息和知识**

NLP有很多典型的应用场景，例如：

- 机器翻译：将一种自然语言转换成另一种自然语言，如谷歌翻译、百度翻译等。
- 问答系统：根据用户提出的问题，从知识库或文本中提取出答案，如小冰、小爱同学等。
- 情感分析：判断文本中所表达的情绪或态度是正面还是负面，如微博、电商评论等。
- 文本摘要：从长文本中抽取出主要内容或生成简洁概括，如新闻摘要、论文摘要等。

NLP实现起来并不容易，它面临着很多难点和挑战，例如：

- 自然语言的多样性和复杂性：不同的语言有不同的规则、结构、习惯和文化背景，同一种语言也有不同的方言、口音和风格。自然语言还存在歧义、隐喻、省略等现象，使得理解起来更加困难。
- 自然语言的动态性和时效性：自然语言会随着时间、地点和场合而变化，新词新义会不断出现，旧词旧义会逐渐消失。因此需要持续更新数据和模型以适应变化。
- 自然语言处理所需的数据量和计算资源：为了训练高效准确的NLP模型，需要大量标注过的数据作为输入。同时也需要强大的计算能力来处理这些数据。这些都给NLP带来了成本上的限制。

[为了实现一个完整有效的NLP系统或应用，通常需要经过以下几个步骤](https://easyai.tech/ai-definition/nlp/)[1](https://easyai.tech/ai-definition/nlp/)：

- 数据收集：从各种来源获取原始数据，并进行清洗、去重、格式化等预处理操作。
- 数据标注：对数据进行人工或半自动地标注，并检查标注质量。标注内容根据具体任务而定，如分词、命名实体识别、情感分类等。
- 模型构建：选择合适的模型结构，并定义损失函数、优化器等参数。模型结构可以是传统的基于规则或统计方法，也可以是现代基于神经网络方法。
- 模型训练：使用标注好的数据对模型进行训练，并监控训练过程中模型在训练集和验证集上表现。如果发现过拟合或欠拟合问题，则调整模型参数或使用正则化技术。
- 模型评估：使用测试集对模型进行评估，并使用相应任务指标衡量模型性能。指标可以是准确率（accuracy）、召回率（recall）、F1值（F1-score）等。
- 模型部署：将训练好的模型部署到实际的应用场景中，并提供接口和服务。同时也要定期更新模型以适应数据和需求的变化。

### 2. 深度学习时代

![](/Users/zhangbo/Downloads/20191019231950281.png)

[自然语言处理领域的范式变迁](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E5%A4%84%E7%90%86%E9%A2%86%E5%9F%9F%E7%9A%84%E8%8C%83%E5%BC%8F%E5%8F%98%E8%BF%81)

- [深度学习时代](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%B7%B1%E5%BA%A6%E5%AD%A6%E4%B9%A0%E6%97%B6%E4%BB%A3)
  - [词向量(Word Embedding)](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E8%AF%8D%E5%90%91%E9%87%8Fword-embedding)
    - [One-hot编码](http://fancyerii.github.io/2023/02/20/about-chatgpt/#one-hot%E7%BC%96%E7%A0%81)
    - [神经网络语言模型](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B)
    - [Word2Vec](http://fancyerii.github.io/2023/02/20/about-chatgpt/#word2vec)
  - [从DNN到CNN再到RNN](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%BB%8Ednn%E5%88%B0cnn%E5%86%8D%E5%88%B0rnn)
    - [CNN在NLP的应用](http://fancyerii.github.io/2023/02/20/about-chatgpt/#cnn%E5%9C%A8nlp%E7%9A%84%E5%BA%94%E7%94%A8)
    - [RNN、LSTM和GRU](http://fancyerii.github.io/2023/02/20/about-chatgpt/#rnnlstm%E5%92%8Cgru)
  - [Seq2Seq和Attention机制](http://fancyerii.github.io/2023/02/20/about-chatgpt/#seq2seq%E5%92%8Cattention%E6%9C%BA%E5%88%B6)
    - [Seq2Seq问题和Encoder-Decoder框架](http://fancyerii.github.io/2023/02/20/about-chatgpt/#seq2seq%E9%97%AE%E9%A2%98%E5%92%8Cencoder-decoder%E6%A1%86%E6%9E%B6)
    - [Attention机制](http://fancyerii.github.io/2023/02/20/about-chatgpt/#attention%E6%9C%BA%E5%88%B6)
  - [Transformer](http://fancyerii.github.io/2023/02/20/about-chatgpt/#transformer)
    - [Transformer解决什么问题](http://fancyerii.github.io/2023/02/20/about-chatgpt/#transformer%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
    - [Self Attention](http://fancyerii.github.io/2023/02/20/about-chatgpt/#self-attention)
    - [Self Attention](http://fancyerii.github.io/2023/02/20/about-chatgpt/#self-attention)
      - [大规模预训练语言模型+微调的时代](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%A4%A7%E8%A7%84%E6%A8%A1%E9%A2%84%E8%AE%AD%E7%BB%83%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E5%BE%AE%E8%B0%83%E7%9A%84%E6%97%B6%E4%BB%A3)
        - [当时存在的问题](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%BD%93%E6%97%B6%E5%AD%98%E5%9C%A8%E7%9A%84%E9%97%AE%E9%A2%98)
          - [监督数据不足](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E7%9B%91%E7%9D%A3%E6%95%B0%E6%8D%AE%E4%B8%8D%E8%B6%B3)
          - [中间任务多](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%B8%AD%E9%97%B4%E4%BB%BB%E5%8A%A1%E5%A4%9A)
        - [解决思路](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E8%A7%A3%E5%86%B3%E6%80%9D%E8%B7%AF)
        - [从ELMo到GPT再到BERT](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%BB%8Eelmo%E5%88%B0gpt%E5%86%8D%E5%88%B0bert)
          - [ELMo](http://fancyerii.github.io/2023/02/20/about-chatgpt/#elmo)
          - [GPT](http://fancyerii.github.io/2023/02/20/about-chatgpt/#gpt)
          - [BERT](http://fancyerii.github.io/2023/02/20/about-chatgpt/#bert)
      - [大规模语言模型+自然语言交互的时代](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%A4%A7%E8%A7%84%E6%A8%A1%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E4%BA%A4%E4%BA%92%E7%9A%84%E6%97%B6%E4%BB%A3)
        - [另辟蹊径](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%8F%A6%E8%BE%9F%E8%B9%8A%E5%BE%84)
        - [GPT-3](http://fancyerii.github.io/2023/02/20/about-chatgpt/#gpt-3)
          - [Zero-shot Learning](http://fancyerii.github.io/2023/02/20/about-chatgpt/#zero-shot-learning)
          - [One-shot Learning](http://fancyerii.github.io/2023/02/20/about-chatgpt/#one-shot-learning)
          - [Few-shot Learning](http://fancyerii.github.io/2023/02/20/about-chatgpt/#few-shot-learning)
          - [和微调的对比](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%92%8C%E5%BE%AE%E8%B0%83%E7%9A%84%E5%AF%B9%E6%AF%94)
          - [模型大小](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%A8%A1%E5%9E%8B%E5%A4%A7%E5%B0%8F)
          - [训练数据](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E8%AE%AD%E7%BB%83%E6%95%B0%E6%8D%AE)
          - [结果和结论](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E7%BB%93%E6%9E%9C%E5%92%8C%E7%BB%93%E8%AE%BA)
        - [Prompt-based Learning](http://fancyerii.github.io/2023/02/20/about-chatgpt/#prompt-based-learning)
          - [基本概念](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5)
          - [Prompt-based Learning是下一代范式吗？](http://fancyerii.github.io/2023/02/20/about-chatgpt/#prompt-based-learning%E6%98%AF%E4%B8%8B%E4%B8%80%E4%BB%A3%E8%8C%83%E5%BC%8F%E5%90%97)
        - [推理、思维链和知识编辑](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%8E%A8%E7%90%86%E6%80%9D%E7%BB%B4%E9%93%BE%E5%92%8C%E7%9F%A5%E8%AF%86%E7%BC%96%E8%BE%91)
          - [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](http://fancyerii.github.io/2023/02/20/about-chatgpt/#chain-of-thought-prompting-elicits-reasoning-in-large-language-models)
          - [Large Language Models are Zero-Shot Reasoners](http://fancyerii.github.io/2023/02/20/about-chatgpt/#large-language-models-are-zero-shot-reasoners)
          - [Locating and Editing Factual Associations in GPT](http://fancyerii.github.io/2023/02/20/about-chatgpt/#locating-and-editing-factual-associations-in-gpt)
    - [ChatGPT原理](http://fancyerii.github.io/2023/02/20/about-chatgpt/#chatgpt%E5%8E%9F%E7%90%86)
      - [以用户为中心](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%BB%A5%E7%94%A8%E6%88%B7%E4%B8%BA%E4%B8%AD%E5%BF%83)
      - [InstructGPT原理](http://fancyerii.github.io/2023/02/20/about-chatgpt/#instructgpt%E5%8E%9F%E7%90%86)
        - [摘要](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%91%98%E8%A6%81)
        - [方法和实验](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%96%B9%E6%B3%95%E5%92%8C%E5%AE%9E%E9%AA%8C)
          - [总体训练流程](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%80%BB%E4%BD%93%E8%AE%AD%E7%BB%83%E6%B5%81%E7%A8%8B)
            - [监督学习微调](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E7%9B%91%E7%9D%A3%E5%AD%A6%E4%B9%A0%E5%BE%AE%E8%B0%83)
            - [训练Reward模型](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E8%AE%AD%E7%BB%83reward%E6%A8%A1%E5%9E%8B)
          - [强化学习微调](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%BC%BA%E5%8C%96%E5%AD%A6%E4%B9%A0%E5%BE%AE%E8%B0%83)
          - [数据集](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%95%B0%E6%8D%AE%E9%9B%86)
          - [任务(Task)](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%BB%BB%E5%8A%A1task)
          - [标注数据收集](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%A0%87%E6%B3%A8%E6%95%B0%E6%8D%AE%E6%94%B6%E9%9B%86)
          - [模型](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E6%A8%A1%E5%9E%8B)
            - [强化学习简介(可跳过)](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%BC%BA%E5%8C%96%E5%AD%A6%E4%B9%A0%E7%AE%80%E4%BB%8B%E5%8F%AF%E8%B7%B3%E8%BF%87)
            - [为什么要使用强化学习](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E4%BD%BF%E7%94%A8%E5%BC%BA%E5%8C%96%E5%AD%A6%E4%B9%A0)
            - [Instruct强化学习的Reward](http://fancyerii.github.io/2023/02/20/about-chatgpt/#instruct%E5%BC%BA%E5%8C%96%E5%AD%A6%E4%B9%A0%E7%9A%84reward)
      - [ChatGPT和InstructGPT的区别](http://fancyerii.github.io/2023/02/20/about-chatgpt/#chatgpt%E5%92%8Cinstructgpt%E7%9A%84%E5%8C%BA%E5%88%AB)
2. NLP在工作中的应用场景的
   
   1、ChatGPT的应用，有些可以基于NLP中间技术产物来使用
   
   ![NLP技术全景图](../images/NLP技术全景图.png)
   
   例如：1、语义相似度  2、信息抽取 3、内容搜索、4、实体识别、5、文本分类
   
   -------------
   
   引用内容

> [关于ChatGPT的思考 - 李理的博客](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%A4%A7%E8%A7%84%E6%A8%A1%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E4%BA%A4%E4%BA%92%E7%9A%84%E6%97%B6%E4%BB%A3)[关于ChatGPT的思考 - 李理的博客](http://fancyerii.github.io/2023/02/20/about-chatgpt/#%E5%A4%A7%E8%A7%84%E6%A8%A1%E8%AF%AD%E8%A8%80%E6%A8%A1%E5%9E%8B%E8%87%AA%E7%84%B6%E8%AF%AD%E8%A8%80%E4%BA%A4%E4%BA%92%E7%9A%84%E6%97%B6%E4%BB%A3)

视频学习资料

> [跟李沐学AI](https://space.bilibili.com/1567748478)
