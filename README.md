# Metric Fairness: Is BERTScore Fair?
This repository contains the code and data for our EMNLP paper [BERTScore is Unfair: On Social Bias in Language Model-Based Metrics for Text Generation](https://txsun1997.github.io/papers/metric_bias.pdf).

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

## Mitigate Metric Bias

We are sorting out the code and data for mitigating metric bias. Watch this repository for the latest updates!

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

