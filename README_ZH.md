[**English**](https://github.com/Tencent/TencentPretrain) | [**中文**](https://github.com/Tencent/TencentPretrain/blob/main/README_ZH.md)

## TencentPretrain：腾讯预训练模型框架

预训练已经成为人工智能技术的重要组成部分，为大量人工智能相关任务带来了显著提升。TencentPretrain是一个用于对文本、图像、语音等模态数据进行预训练和微调的工具包。TencentPretrain遵循模块化的设计原则。通过模块的组合，用户能迅速精准的复现已有的预训练模型，并利用已有的接口进一步开发更多的预训练模型。通过TencentPretrain，我们建立了一个模型仓库，其中包含不同性质的预训练模型（例如基于不同模态、编码器、目标任务）。用户可以根据具体任务的要求，从中选择合适的预训练模型使用。TencentPretrain继承了开源项目UER (https://github.com/dbiir/UER-py/) 的部分工作，并在其基础上进一步开发，形成支持多模态的预训练模型框架。

#### **项目使用文档：https://github.com/Tencent/TencentPretrain/wiki/主页**

<br>

目录
=================
  * [项目特色](#项目特色)
  * [依赖环境](#依赖环境)
  * [快速上手](#快速上手)
  * [预训练数据](#预训练数据)
  * [下游任务数据集](#下游任务数据集)
  * [预训练模型仓库](#预训练模型仓库)
  * [使用说明](#使用说明)
  * [竞赛解决方案](#竞赛解决方案)
  * [引用](#引用)

<br>

## 项目特色
TencentPretrain有如下几方面优势:
- __可复现__ TencentPretrain已在许多数据集上进行了测试，与原始预训练模型实现（例如BERT、GPT-2、ELMo、T5、CLIP）的表现相匹配
- __模块化__ TencentPretrain使用解耦的模块化设计框架。框架分成Embedding、Encoder、Target等多个部分。各个部分之间有着清晰的接口并且每个部分包括了丰富的模块。可以对不同模块进行组合，构建出性质不同的预训练模型
- __多模态__ TencentPretrain支持文本、图像、语音模态的预训练模型，并支持模态之间的翻译、融合等操作
- __模型训练__ TencentPretrain支持CPU、单机单GPU、单机多GPU、多机多GPU训练模式，并支持使用DeepSpeed优化库进行超大模型训练
- __模型仓库__ 我们维护并持续发布预训练模型。用户可以根据具体任务的要求，从中选择合适的预训练模型使用
- __SOTA结果__ TencentPretrain支持全面的下游任务，包括文本/图像分类、序列标注、阅读理解、语音识别等，并提供了多个竞赛获胜解决方案
- __预训练相关功能__ TencentPretrain提供了丰富的预训练相关的功能和优化，包括特征抽取、近义词检索、预训练模型转换、模型集成、文本生成等

<br>

## 依赖环境
* Python >= 3.6
* torch >= 1.1
* six >= 1.12.0
* argparse
* packaging
* regex
* 如果使用混合精度，需要安装英伟达的apex
* 如果涉及到TensorFlow模型的转换，需要安装TensorFlow
* 如果在tokenizer中使用sentencepiece模型，需要安装[SentencePiece](https://github.com/google/sentencepiece)
* 如果使用模型集成stacking，需要安装LightGBM和[BayesianOptimization](https://github.com/fmfn/BayesianOptimization)
* 如果使用全词遮罩（whole word masking）预训练，需要安装分词工具，例如[jieba](https://github.com/fxsjy/jieba)
* 如果在序列标注下游任务中使用CRF，需要安装[pytorch-crf](https://github.com/kmkurn/pytorch-crf)
* 如果使用超大模型，需要安装[DeepSpeed](https://github.com/microsoft/DeepSpeed)
* 如果涉及图像模型，需要安装torchvision
* 如果涉及语音模型，需要安装torchaudio，在使用specaugment进行数据增强时部分设置会用到opencv-python

<br>

## 快速上手
这里我们通过常用的例子来简要说明如何使用TencentPretrain，更多的细节请参考使用说明章节。我们首先使用文本预训练模型BERT和书评情感分类数据集。我们在书评语料上对模型进行预训练，然后在书评情感分类数据集上对其进行微调。这个过程有三个输入文件：书评语料，书评情感分类数据集和中文词典。这些文件均为UTF-8编码，并被包括在这个项目中。

BERT模型要求的预训练语料格式是一行一个句子，不同文档使用空行分隔，如下所示：

```
doc1-sent1
doc1-sent2
doc1-sent3

doc2-sent1

doc3-sent1
doc3-sent2
```
书评语料是由书评情感分类数据集去掉标签得到的。我们将一条评论从中间分开，从而形成一个两句话的文档，具体可见*corpora*文件夹中的*book_review_bert.txt*。

分类数据集的格式如下：
```
label    text_a
1        instance1
0        instance2
1        instance3
```
标签和文本之间用\t分隔，第一行是列名。对于n分类，标签应该是0到n-1之间（包括0和n-1）的整数。

词典文件的格式是一行一个单词，我们使用谷歌提供的包含21128个中文字符的词典文件*models/google_zh_vocab.txt*。

我们首先对书评语料进行预处理。预处理阶段需要将语料处理成为指定预训练模型要求的数据格式（*--data_processor*）：
```
python3 preprocess.py --corpus_path corpora/book_review_bert.txt --vocab_path models/google_zh_vocab.txt \
                      --dataset_path dataset.pt --processes_num 8 --data_processor bert
```
注意我们需要安装 *six>=1.12.0*。

预处理非常耗时，使用多个进程可以大大加快预处理速度（*--processes_num*）。默认的分词器为 *--tokenizer bert* 。原始文本在预处理之后被转换为*pretrain.py*可以接收的输入，*dataset.pt*。然后下载Google中文预训练模型[*google_zh_model.bin*](https://share.weiyun.com/FR4rPxc4)（此文件为TencentPretrain支持的格式，原始模型来自于[这里](https://github.com/google-research/bert)），并将其放在 *models* 文件夹中。接着加载Google中文预训练模型，在书评语料上对其进行增量预训练。预训练模型通常由词向量层，编码层和目标任务层组成。因此要构建预训练模型，我们应该给出这些信息，比如编码层使用什么类型的Encoder模块。这里我们通过配置文件（*--config_path*）指定模型使用的模块类型和超参数等信息。具体可见*models/bert/base_config.json*。假设我们有一台带有8个GPU的机器：
```
python3 pretrain.py --dataset_path dataset.pt --vocab_path models/google_zh_vocab.txt \
                    --pretrained_model_path models/google_zh_model.bin \
                    --config_path models/bert/base_config.json \
                    --output_model_path models/book_review_model.bin \
                    --world_size 8 --gpu_ranks 0 1 2 3 4 5 6 7 \
                    --total_steps 5000 --save_checkpoint_steps 1000 --batch_size 32

mv models/book_review_model.bin-5000 models/book_review_model.bin
```
请注意，*pretrain.py*输出的模型会带有记录训练步数的后缀（*--total_steps*），这里我们可以删除后缀以方便使用。

然后，我们在下游分类数据集上微调预训练模型，我们使用 *pretrain.py* 的输出[*book_review_model.bin*](https://share.weiyun.com/PnxMrRwZ)（加载词向量层和编码层参数）：
```
python3 finetune/run_classifier.py --pretrained_model_path models/book_review_model.bin \
                                   --vocab_path models/google_zh_vocab.txt \
                                   --config_path models/bert/base_config.json \
                                   --train_path datasets/book_review/train.tsv \
                                   --dev_path datasets/book_review/dev.tsv \
                                   --test_path datasets/book_review/test.tsv \
                                   --epochs_num 3 --batch_size 32
``` 

微调后的模型的默认路径是*models/finetuned_model.bin*。注意到预训练的实际batch size大小是 *--batch_size* 乘以 *--world_size*；分类等下游任务微调的实际的batch size大小是 *--batch_size* 。然后我们利用微调后的分类器模型进行预测：
```
python3 inference/run_classifier_infer.py --load_model_path models/finetuned_model.bin \
                                          --vocab_path models/google_zh_vocab.txt \
                                          --config_path models/bert/base_config.json \
                                          --test_path datasets/book_review/test_nolabel.tsv \
                                          --prediction_path datasets/book_review/prediction.tsv \
                                          --labels_num 2
```
*--test_path* 指定需要预测的文件，文件需要包括text_a列；
*--prediction_path* 指定预测结果的文件；
注意到我们需要指定分类任务标签的个数 *--labels_num* ，这里是二分类任务。

<br>

以上我们给出了用TencentPretrain进行预处理、预训练、微调、推理的基本使用方式。更多的例子参见完整的 :arrow_right: [__快速上手__](https://github.com/Tencent/TencentPretrain/wiki/快速上手) :arrow_left: 。完整的快速上手包含了全面的例子，覆盖了大多数预训练相关的使用场景。推荐用户完整阅读快速上手章节，以便能合理的使用本项目。

<br>

## 预训练数据
我们提供了链接，指向一系列开源的 :arrow_right: [__预训练数据__](https://github.com/Tencent/TencentPretrain/wiki/预训练数据) :arrow_left: 。

<br>

## 下游任务数据集
我们提供了链接，指向一系列开源的 :arrow_right: [__下游任务数据集__](https://github.com/Tencent/TencentPretrain/wiki/下游任务数据集) :arrow_left: 。TencentPretrain可以直接加载这些数据集。

<br>

## 预训练模型仓库
借助TencentPretrain，我们训练不同性质的预训练模型（例如基于不同模态、编码器、目标任务）。用户可以在 :arrow_right: [__预训练模型仓库__](https://github.com/Tencent/TencentPretrain/wiki/预训练模型仓库) :arrow_left: 中找到各种性质的预训练模型以及它们对应的描述和下载链接。所有预训练模型都可以由TencentPretrain直接加载。将来我们会发布更多的预训练模型。

<br>

## 使用说明
TencentPretrain使用解耦的设计框架，方便用户使用和扩展，项目组织如下：
```
TencentPretrain/
    |--tencentpretrain/
    |    |--embeddings/ # 包括词向量模块
    |    |--encoders/ # 包括编码器模块，例如RNN, CNN, Transformer
    |    |--decoders/ # 包括解码器模块
    |    |--targets/ # 包括目标任务模块，例如语言模型, 遮罩语言模型
    |    |--layers/ # 包括常用的神经网络层
    |    |--models/ # 包括 model.py，用于组合词向量（embedding）、编码器（encoder）、目标任务（target）等模块
    |    |--utils/ # 包括常用的功能模块
    |    |--model_builder.py
    |    |--model_loader.py
    |    |--model_saver.py
    |    |--opts.py
    |    |--trainer.py
    |
    |--corpora/ # 预训练数据存放文件夹
    |--datasets/ # 下游任务数据集存放文件夹
    |--models/ # 模型、词典、配置文件存放文件夹
    |--scripts/ # 实用脚本存放文件夹
    |--finetune/ # 微调脚本存放文件夹
    |--inference/ # 前向推理脚本存放文件夹
    |
    |--preprocess.py
    |--pretrain.py
    |--README.md
    |--README_ZH.md
    |--requirements.txt
    |--LICENSE

```

在 :arrow_right: [__使用说明__](https://github.com/Tencent/TencentPretrain/wiki/使用说明) :arrow_left: 中给出了全面的TencentPretrain使用示例。这些示例可以帮助用户快速实现BERT、GPT-2、ELMo、T5、CLIP等预训练模型以及使用这些预训练模型在一系列下游任务上进行微调。

<br>

## 竞赛解决方案
TencentPretrain已被用于许多竞赛的获奖解决方案中。在本章节中，我们提供了一些使用TencentPretrain在竞赛中获得SOTA成绩的示例，例如CLUE。更多详细信息参见 :arrow_right: [__竞赛解决方案__](https://github.com/Tencent/TencentPretrain/wiki/竞赛解决方案) :arrow_left: 。

<br/>

## 引用
#### 如果您在您的学术工作中使用我们的工作（比如模型仓库中的预训练模型），可以引用我们的[论文](https://arxiv.org/pdf/2212.06385.pdf)：
```
@article{zhao2023tencentpretrain,
  title={TencentPretrain: A Scalable and Flexible Toolkit for Pre-training Models of Different Modalities},
  author={Zhao, Zhe and Li, Yudong and Hou, Cheng and Zhao, Jing and others},
  journal={ACL 2023},
  pages={217},
  year={2023}
}
```
