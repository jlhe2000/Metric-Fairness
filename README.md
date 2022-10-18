# Metric Fairness: Is BERTScore Fair?
This repository contains the code and data for our EMNLP paper [BERTScore is Unfair: On Social Bias in Language Model-Based Metrics for Text Generation](https://arxiv.org/abs/2210.07626).

## Overview

Pre-trained language model-based metrics (PLM-based metrics, e.g., BERTScore, MoverScore, BLEURT) have been widely used in various text generation tasks including machine translation, text summarization, etc. Compared with traditional $n$-gram-based metrics (e.g., BLEU, ROUGE, NIST), PLM-based metrics can well capture the semantic similarity between system outputs and references, and therefore achieve higher correlation with human judgements. However, it is well known that PLMs can encode a high degree of social bias. How much of the social bias in BERT is inherited by BERTScore? Will BERTScore encourage systems that generate biased text? To what extent do different PLM-based metrics carry social bias? This work presents the first systematic study on social bias in PLM-based metrics for text generation.

The following figure illustrates the impact of social bias in PLM-based metrics: If the metric is biased against some sensitive attributes (e.g., gender), generative models that express such bias will be rewarded and selected. The texts generated by these biased models may be incorporated in the corpus, further reinforcing the social bias in data. 


![](https://github.com/txsun1997/Metric-Fairness/blob/main/metric-bias.png)

Our work includes:

1. **Measuring social bias in PLM-based metrics.** We constructed datasets and metrics to measure the bias (or unfairness) in existing PLM-based metrics including BERTScore, MoverScore, BLEURT, PRISM, BARTScore, and FrugalScore.
2. **Mitigating social bias in PLM-based metrics.** We explore mitigating social bias in existing metrics by (1) replacing the backbone models with debiased ones such as [Zari models](https://github.com/google-research-datasets/Zari) and (2) training debiasing adapters on augmented data.

## Measure Metric Bias

We have uploaded our constructed datasets for measuring metric bias (see [/measuring_bias/data](/measuring_bias/data)). We provide 6 datasets for evaluating social bias against different sensitive attributes including age, gender, physical appearance, race, religion, and socioeconomic status. We also provide our evaluated scores using 29 existing text generation metrics for each sample in the datasets. You can reproduce our results as follows:

```bash
pip install prettytable
git clone https://github.com/txsun1997/Metric-Fairness
cd Metric-Fairness/measuring_bias
python get_bias_score.py
```

If all is well, you should obtain the following results:

```
+-------------------------+-------+--------+---------------------+------+----------+---------------+
|          metric         |  age  | gender | physical-appearance | race | religion | socioeconomic |
+-------------------------+-------+--------+---------------------+------+----------+---------------+
|  bartscore-bart-base-f  |  6.2  |  3.67  |         6.04        | 2.44 |   5.97   |      6.65     |
|  bartscore-bart-base-p  |  6.51 |  6.5   |         7.59        | 2.6  |   7.63   |      8.0      |
|  bartscore-bart-base-r  |  7.1  |  2.47  |         8.44        | 2.52 |   7.12   |      7.55     |
|  bartscore-bart-large-f |  3.83 |  9.47  |         6.38        | 1.67 |   4.7    |      3.47     |
|  bartscore-bart-large-p |  7.65 | 14.17  |         6.42        | 1.87 |   5.13   |      4.55     |
|  bartscore-bart-large-r |  2.36 |  3.69  |         4.92        | 2.13 |   4.34   |      3.48     |
|   bertscore-bert-base   |  5.68 |  8.73  |         6.36        | 1.24 |   6.2    |      7.66     |
|   bertscore-bert-large  |  4.64 |  4.39  |         6.07        | 2.3  |   7.87   |      6.85     |
|   bertscore-distilbert  |  5.26 |  8.36  |         4.93        | 1.94 |   6.82   |      7.64     |
|  bertscore-roberta-base |  6.63 |  3.75  |         7.82        | 2.27 |   4.08   |      6.21     |
| bertscore-roberta-large |  8.23 |  6.99  |         7.94        | 2.59 |   4.64   |      7.4      |
|           bleu          |  2.35 |  0.1   |         0.94        | 0.19 |   0.61   |      2.79     |
|     bleurt-bert-base    | 13.44 | 29.97  |        12.92        | 3.02 |  16.21   |     15.41     |
|    bleurt-bert-large    | 15.07 | 27.08  |         7.98        | 4.0  |  16.18   |      14.6     |
|     bleurt-bert-tiny    | 14.01 |  6.47  |        10.71        | 8.43 |   6.39   |     13.01     |
|      bleurt-rembert     | 16.52 | 20.93  |         8.84        | 4.21 |  17.12   |     12.93     |
|           chrf          |  3.43 |  1.23  |         1.57        | 1.89 |   1.44   |      3.46     |
| frugalscore-bert-medium |  5.02 |  5.73  |         5.07        | 0.93 |   5.57   |      8.09     |
|  frugalscore-bert-small |  4.9  |  7.04  |         4.64        | 0.91 |   5.82   |      8.78     |
|  frugalscore-bert-tiny  |  7.96 |  3.2   |         5.27        | 1.39 |   5.96   |      7.12     |
|          meteor         |  4.96 |  2.63  |         3.08        | 1.53 |   2.56   |      4.4      |
|   moverscore-bert-base  |  6.06 | 11.36  |         6.69        | 3.84 |   9.63   |      7.94     |
|  moverscore-bert-large  |  6.78 |  6.68  |         8.04        | 4.43 |  10.24   |      8.3      |
|  moverscore-distilbert  |  7.24 | 13.24  |         4.94        | 3.35 |   9.67   |      8.59     |
|           nist          |  2.2  |  0.11  |         1.03        | 0.25 |   0.54   |      1.43     |
|         prism-f         |  6.69 |  7.13  |         7.48        | 1.97 |   6.79   |      4.85     |
|         prism-p         |  9.1  | 14.33  |         7.05        | 2.6  |   7.06   |      6.51     |
|         prism-r         |  5.1  |  3.0   |         7.13        | 2.65 |   5.92   |      4.91     |
|          rouge          |  3.83 |  0.21  |         2.01        | 0.12 |   1.02   |      3.4      |
+-------------------------+-------+--------+---------------------+------+----------+---------------+
```

You can also evaluate the bias types (and text generation metrics) of interest instead of all of them by specifying the parameters `--bias_type` and `--metric_name`. For example,

```bash
python get_bias_score.py --bias_type age gender --metric_name rouge bleu
```

would result in a tiny table:

```
+--------+------+--------+
| metric | age  | gender |
+--------+------+--------+
|  bleu  | 2.35 |  0.1   |
| rouge  | 3.83 |  0.21  |
+--------+------+--------+
```

You can see the exact details of how we calculated each score by checking `metrics.py`, and if you need to calculate scores of default backbones, run

```bash
cd Metric-Fairness/measuring_bias/metrics
pip install -r requirements.txt
bash metrics.sh
```

and you will obtain an output file named `scores.csv` by default which contains scores on our gender bias dataset.

Also, you can calculate the polarized bias scores by running

```bash
python cal_bias_score.py --polarity True
```
You are expected to obtain the results reported in Table 9 in the paper.

Below is an example output of `python cal_bias_score.py` (without polarity):

```
+------+-------+--------+------+------+-------------+-------------+-------------+------------+--------+---------+---------+---------+-------------+-------------+-------------+-------------+
| bleu | rouge | meteor | nist | chrf | bertscore_r | bertscore_p | bertscore_f | moverscore | bleurt | prism_r | prism_p | prism_f | bartscore_r | bartscore_p | bartscore_f | frugalscore |
+------+-------+--------+------+------+-------------+-------------+-------------+------------+--------+---------+---------+---------+-------------+-------------+-------------+-------------+
| 0.1  |  0.21 |  2.14  | 0.12 | 1.23 |     4.61    |     9.04    |     6.99    |   13.24    |  30.0  |   3.0   |  14.33  |   7.13  |     3.69    |    14.17    |     9.47    |     3.19    |
+------+-------+--------+------+------+-------------+-------------+-------------+------------+--------+---------+---------+---------+-------------+-------------+-------------+-------------+
```



## Mitigate Metric Bias

### Train

#### Datasets

[Download link](https://drive.google.com/drive/folders/1rqPw_h6_0CxgL4LY2LhBPMR2RODnMnv6?usp=sharing)

We collect training data based on two public sentence-pair datasets, MultiNLI [(Williams et al., 2018)](https://doi.org/10.18653/v1/n18-1101) and STS-B [(Cer et al., 2017)](http://arxiv.org/abs/1708.00055), in which each sample is comprised of a premise and a hypothesis. We perform counterfactual data augmentation (CDA) ([Zhao et al., 2018b)](https://arxiv.org/abs/1804.06876) on the sentences in MultiNLI and STS-B to construct a training set. You can download the datasets from the above link, which includes `train.tsv` for BERTScore (both BERT-base and BERT-large), BARTScore (BART-base), and BLEURT (BERT-base).

The following example shows how to add and train a debiasing adapter on the BERT-large model of BERTScore. Note that we used a single NVIDIA 3090 GPU (24GB) to perform training.

```bash
cd Metric-Fairness/mitigating_bias/train/BERTScore
mkdir ./logs
pip install -r requirements.txt
INPUT_PATH=train.tsv # your training set path
python train_BERTScore.py
    --model_type bert-large-uncased \
    --adapter_name debiased-bertscore \
    --lr 5e-4 \
    --warmup 0.0 \
    --batch_size 16 \
    --n_epochs 4 \
    --seed 42 \
    --device cuda \
    --logging_steps 100 \
    --data_path ${INPUT_PATH}
```

After training, a debiasing adapter will be saved in `./adapter/`, and you can check more training details in `./logs` . See also [fitlog](https://fitlog.readthedocs.io/zh/latest/).

### Test

#### Adapters

[Download link](https://drive.google.com/drive/folders/1nqTQWXtf14SXZ5pC0hK5kBa28q9h-0-y?usp=sharing)

We have trained debiasing adapters for BERTScore (both BERT-base and BERT-large), BARTScore (BART-base), and BLEURT (BERT-base), so you can download these adapters' checkpoints through the link above.

The following example shows how to add our trained debiasing adapters to BERTScore (both BERT-base and BERT-large), BARTScore (BART-base), and BLEURT (BERT-base) , and calculate the bias scores using debiased metrics on our test set in `Metric-Fairness/mitigating_bias/test/test_data` [(WinoBias)](https://doi.org/10.18653/v1/n18-2003).

```bash
cd Metric-Fairness/mitigating_bias/test
pip install -r requirements.txt
BERT_SCORE_BERT_LARGE_ADAPTER_PATH=BERTScore/BERT-large/adapter # bert_score_bert_large adapter path
BERT_SCORE_BERT_BASE_ADAPTER_PATH=BERTScore/BERT-base/adapter # bert_score_bert_base adapter path
BLEURT_BERT_BASE_ADAPTER_PATH=BLEURT/adapter # bleurt_bert_base adapter path
BART_SCORE_BART_BASE_ADAPTER_PATH=BARTScore/adapter # bart_score_bart_base adapter path
python cal_debias_scores.py
    --bert_score_bert_large_adapter_path ${BERT_SCORE_BERT_LARGE_ADAPTER_PATH} \
    --bert_score_bert_base_adapter_path ${BERT_SCORE_BERT_BASE_ADAPTER_PATH} \
    --bleurt_bert_base_adapter_path ${BLEURT_BERT_BASE_ADAPTER_PATH} \
    --bart_score_bart_base_adapter_path ${BART_SCORE_BART_BASE_ADAPTER_PATH}
```

Below is an example result:

```
+----------------------+-----------------------+------------------+----------------------+
| bert_score_bert_base | bert_score_bert_large | bleurt_bert_base | bart_score_bart_base |
+----------------------+-----------------------+------------------+----------------------+
|         4.21         |          2.69         |      10.46       |         2.35         |
+----------------------+-----------------------+------------------+----------------------+
```

### Performance Evaluation

## Citation

If you use our data or code, please cite:

```bibtex
@inproceedings{sun2022bertscore,
  title={BERTScore is Unfair: On Social Bias in Language Model-Based Metrics for Text Generation},
  author={Tianxiang Sun and Junliang He and Xipeng Qiu and Xuanjing Huang},
  booktitle = {Proceedings of {EMNLP}},
  year={2022}
}
```

