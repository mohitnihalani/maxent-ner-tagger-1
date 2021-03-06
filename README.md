# Maximum-entropy NER tagger

A [GitHub repository for this project](https://github.com/melanietosik/maxent-ner-tagger) is available online.

## Overview

The goal of this project was to implement and train a [named-entity recognizer (NER)](https://en.wikipedia.org/wiki/Named-entity_recognition). Most of the feature builder functionality was implemented using [spaCy](https://spacy.io/), an industrial-strength, open-source NLP library written in Python/Cython. For classification a maximum entropy (MaxEnt) classifier is used.

## Implementation details

The dataset for this task is the [2003 CoNLL (Conference on Natural Language Learning)](https://aclweb.org/anthology/W/W03/W03-0419.pdf) corpus, which is primarily composed of Reuters news data. The data files are pre-preprocessed and already contain one token per line, its part-of-speech (POS) tag, a BIO (short for beginning, inside, outside) chunk tag, and the corresponding NER tag. 

SpaCy's built-in [token features](https://spacy.io/api/token) proved most useful for feature engineering. Making use of external word lists, such as the Wikipedia gazetteers distributed as part of the [Illinois Named Entity Tagger](https://cogcomp.org/page/software_view/NETagger), generally lead to a decrease in tagging accuracy. Since the data files are relatively large, the gazetteer source code and files are not included in the final submission. I also experimented with boosting model performance by including the prior state/tag as a feature. Somewhat surprisingly, model performance largely remained unchanged, presumably due to the fact that each label is predicted from the same feature set that is encoded in the model anyway.

A [multinomial logistic regression](https://en.wikipedia.org/wiki/Multinomial_logistic_regression) classifier is used for classification. Specifically, I used the ["Logistic Regression" classifier implemented in scikit-learn](http://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html). For a proof of mathematical equivalency, please see ["The equivalence of logistic regression and maximum entropy models"](http://www.win-vector.com/dfiles/LogisticRegressionMaxEnt.pdf) (Mount, 2011).

To make I/O processes a little more uniform, I converted the original data files for the training, development, and test split to a single `.pos-chunk-name` file per split, each containing one token per line and the POS, BIO, and name tags. The conformed data files can be found in the [`CoNLL/`](https://github.com/melanietosik/maxent-ner-tagger/tree/master/CoNLL) data directory.

## Running the code

The NER tagger is implemented in [`scripts/name_tagger.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/name_tagger.py). With each run, the programm will extract features for the training, development, and test split. All feature files are written to the [`CoNNL/`](https://github.com/melanietosik/maxent-ner-tagger/tree/master/CoNLL) data directory with a `.feat.csv` extension. The `.csv` files are compact and readable and can be manually inspected.

After model training, the program also writes a serialized model and vectorizer object to the [`data/`](https://github.com/melanietosik/maxent-ner-tagger/tree/master/data) directory. The vectorizer object is a pickled dump of scikit-learn's ["DictVectorizer"](http://scikit-learn.org/stable/modules/generated/sklearn.feature_extraction.DictVectorizer.html) fit to the training data, which is used to transform the lists of feature-value mappings to binary (one-hot) vectors. In other words, one boolean-valued feature is constructed for each of the possible string values that a feature can take on. For example, a feature ``is_stop`` that can either be `"True"` or `"False"` will become two features in the output, one for `"is_stop=True"` and one for `"is_stop=False"`.

Finally, the program writes two tagged `.name` files to the [`output/`](https://github.com/melanietosik/maxent-ner-tagger/tree/master/output) directory, one for the development set ([`dev.name`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/output/dev.name)) and one for the test set ([`test.name`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/output/test.name)). All settings can be adjusted by editing the paths specified in [`scripts/settings.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/settings.py).

### Installation requiremements

Before running the code, you need to create a [virtual environment](https://virtualenv.pypa.io/en/stable/) and install the required Python dependencies:

```
[maxent-ner-tagger]$ virtualenv -p python3 env
[maxent-ner-tagger]$ source env/bin/activate
(env) [maxent-ner-tagger]$ pip install -U spacy
(env) [maxent-ner-tagger]$ python -m spacy download en  # Download spaCy's English language model files
(env) [maxent-ner-tagger]$ pip install -r requirements.txt
```

### Run the tagger

To **(re-)run the tagger**, in the root directory of the project, run:

```
(env) [maxent-ner-tagger]$ python scripts/name_tagger.py
```

You should start seeing output pretty much immediately. It takes about 5 minutes to regenerate the features and retrain the model. Please note that **all output files will be over-written** with each run.

```
(env) [maxent-ner-tagger]$ python scripts/name_tagger.py
Generating train features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000
Lines processed:    60000
Lines processed:    70000
Lines processed:    80000
Lines processed:    90000
Lines processed:   100000
Lines processed:   110000
Lines processed:   120000
Lines processed:   130000
Lines processed:   140000
Lines processed:   150000
Lines processed:   160000
Lines processed:   170000
Lines processed:   180000
Lines processed:   190000
Lines processed:   200000
Lines processed:   210000

Writing output file: CoNLL/CONLL_train.pos-chunk-name.feat.csv

Generating dev features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000

Writing output file: CoNLL/CONLL_dev.pos-chunk-name.feat.csv

Generating test features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000

Writing output file: CoNLL/CONLL_test.pos-chunk-name.feat.csv

Training model...
X (219554, 449494)
y (219554,)

Tagging dev...
X (55044, 449494)
y (55044,)

Writing output file: output/dev.name

Tagging test...
X (50350, 449494)
y (50350,)

Writing output file: output/test.name

Scoring development set...
50523 out of 51578 tags correct
  accuracy: 97.95
5917 groups in key
6006 groups in response
5160 correct groups
  precision: 85.91
  recall:    87.21
  F1:        86.56

python scripts/name_tagger.py  307.34s user 20.51s system 105% cpu 5:11.00 total
```

## Results

By default, the model always makes use of the POS and BIO tags that are provided with the CoNLL data sets. In addition, I experimented with the following groups of token features:

- `idx`
- `is_alpha`, `is_ascii`, `is_digit`, `is_punct`, `like_num`
- `is_title`, `is_lower`, `is_upper`
- `orth_`, `lemma_`, `lower_`, `norm_`
- `shape_`
- `prefix_`, `suffix_`
- `pos_`, `tag_`, `dep_`
- `is_stop`, `cluster`
- `head`, `left_edge`, `right_edge`

Most of features should be self-explanatory. For detailed descriptions, please see the documentation in [`scripts/name_tagger.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/name_tagger.py) or spaCy's [Token API documentation](https://spacy.io/api/token). Since only one sentence is processed at a time, I decided to also include _context token_ features. Currently, the model uses the token features of `n=2` tokens to the left and the right of the target token.

**Note:** I slightly modified the `scorer.name.py` script so it could be easily be imported as a module and run on Python 3. No changes were made to the actual scoring functionality. The revised script is stored in [`scripts/scorer.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/scorer.py).

### Development set

A breakdown of the experimental feature sets and corresponding _group accuracies_ on the development set is provided below. For the full log of results, please see [`log.md`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/log.md).

| Feature set                                                                                                                                |                            Accuracy |
|:-------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------:|
| CoNLL only                                                                                                                                 | P: 66.77<br> R: 71.46<br> **F1: 69.03** |
| CoNLL + `is_title`, `orth_`                                                                                                                | P: 70.93<br> R: 72.74<br> **F1: 71.82** |
| CoNLL + `idx`, `is_title`, `orth_`                                                                                                         | P: 69.45<br> R: 72.84<br> **F1: 71.10** |
| CoNLL + `is_alpha`, `is_ascii`, `is_digit`, `is_punct`, `is_title`, `like_num`, `orth_`                                                    | P: 68.66<br> R: 70.36<br> **F1: 69.50** |
| CoNLL + `is_lower`, `is_title`, `is_upper`, `orth_`                                                                                        | P: 69.99<br> R: 73.79<br> **F1: 71.84** |
| CoNLL + `is_title`, `lemma_`, `orth_`                                                                                                      | P: 73.17<br> R: 76.14<br> **F1: 74.62** |
| CoNLL + `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`                                                                                   | P: 73.37<br> R: 76.83<br> **F1: 75.06** |
| CoNLL + `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `shape_`                                                                         | P: 72.94<br> R: 78.77<br> **F1: 75.75** |
| CoNLL + `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `prefix_`, `shape_`, `suffix_`                                                   | P: 71.51<br> R: 78.54<br> **F1: 74.86** |
| CoNLL + `dep_`, `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `pos_`, `shape_`, `tag_`                                                 | P: 72.89<br> R: 79.80<br> **F1: 76.19** |
| CoNLL + `cluster`, `dep_`, `is_stop`, `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `pos_`, `shape_`, `tag_`                           | P: 70.32<br> R: 78.38<br> **F1: 74.13** |
| CoNLL + `dep_`, `head`, `is_title`, `left_edge`, `lemma_`, `lower_`, `norm_`, `orth_`, `pos_`, `right_edge`, `shape_`, `tag_`              | P: 78.76<br> R: 83.37<br> **F1: 81.00** |
| CoNLL + `dep_`, `head`, `is_title`, `left_edge`, `lemma_`, `lower_`, `norm_`, `orth_`, `pos_`, `right_edge`, `shape_`, `tag_` + context    | P: 85.64<br> R: 87.07<br> **F1: 86.35** |
| CoNLL + `head`, `is_title`, `left_edge`, `lemma_`, `lower_`, `norm_`, `orth_`, `right_edge`, `shape_` + context                            | P: 85.90<br> R: 86.92<br> **F1: 86.41** |
| CoNLL + `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `shape_`] + context                                                              | P: 85.75<br> R: 86.51<br> **F1: 86.13** |
| **CoNLL + `is_title`, `lemma_`, `lower_`, `norm_`, `orth_`, `prefix_`, `shape_`, `suffix_` + context**                                     | P: 85.91<br> R: 87.21<br> **F1: 86.56** |
| CoNLL + `is_title`, `lemma_`, `orth_`, `prefix_`, `shape_`, `suffix_` + context                                                            | P: 85.80<br> R: 86.99<br> **F1: 86.39** |

Overall, I found that the word itself and its various normalized word forms (exact, lemma, lowercased, normalized) were very helpful indicators for NER tagging. In addition, including whether or not the token was written in title case and its general shape (e.g. `Xxxx` or `dd`) improved model performance as well. Finally, I was able to achieve a **F1 score of 86.56** on the development set by incorporating context token features as well.

### Test set

The tagged output file for the test is written to [`output/test.name`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/output/test.name). To score the test file directly after model training, make sure the tagged gold file exists at `CoNLL/CONLL_test.name` and comment out the relevant lines below `"# Get score on test data"` in [`scripts/name_tagger.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/name_tagger.py). Otherwise you can just use the original scorer script directly with the tagged output file.

## Word embeddings

As a follow-up task, I worked on enhancing the named-entity tagger by adding [word embeddings](https://en.wikipedia.org/wiki/Word_embedding) as input features to the classifier as well. Specifically, I made use of a [publicly available set of GloVe vectors](https://nlp.stanford.edu/projects/glove/) to obtain vector representations for each token in the CoNLL dataset.

### Gensim, GloVe, and word2vec

[Gensim](https://radimrehurek.com/gensim/) is a robust, open-source vector space modeling toolkit implemented in Python. To install, run:

```
(env) [maxent-ner-tagger]$ pip install gensim
```

In order to use GloVe vectors with Gensim, we first have to convert the GloVe vectors to the [word2vec](https://code.google.com/archive/p/word2vec/) format it expects. Gensim provides a convencience script called [`glove2word2vec`](https://radimrehurek.com/gensim/scripts/glove2word2vec.html) to do just that:

```
(env) [maxent-ner-tagger]$ python -m gensim.scripts.glove2word2vec --input data/glove/glove.6B.50d.txt --output data/word2vec/word2vec.6B.50d.txt
2018-04-03 15:45:37,757 - glove2word2vec - INFO - running /Users/.../env/lib/python3.6/site-packages/gensim/scripts/glove2word2vec.py --input data/glove/glove.6B.50d.txt --output data/word2vec/word2vec.6B.50d.txt
2018-04-03 15:45:37,987 - glove2word2vec - INFO - converting 400000 vectors from data/glove/glove.6B.50d.txt to data/word2vec/word2vec.6B.50d.txt
2018-04-03 15:45:38,660 - glove2word2vec - INFO - Converted model with 400000 vectors and 50 dimensions
```

Since the input and output files are relatively large, only the Glove and word2vec files for 100-dimensional vectors are included in the submission. All other word2vec files can easily be regenerated as shown above.

### Run

The program can be run the same way as before:

```
(env) [maxent-ner-tagger]$ python scripts/name_tagger.py
```

The number of dimensions is specified as a global variable at the top of the [`scripts/name_tagger.py`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/scripts/name_tagger.py) file. In addition, there is a flag to enable (mini-batch) [K-means clustering](http://scikit-learn.org/stable/modules/generated/sklearn.cluster.MiniBatchKMeans.html), which will cluster the high-dimensional word vectors into `k` clusters during training, and predict the closest cluster ID for each token vector at test time. I also implemented binarization of embeddings following [Guo et al. (2014)](http://aclweb.org/anthology/D/D14/D14-1012.pdf) but observed a significant drop in accuracy when using discrete binarization features instead of continuous word embeddings. Binarization is disabled in the current version of the name tagger.

Make sure you generate a `data/word2vec/` directory that contains at least one of the following files (`word2vec.6B.100d.txt`) before you run the program:

```
(env) [maxent-ner-tagger]$ ls data/word2vec
word2vec.6B.100d.txt word2vec.6B.200d.txt word2vec.6B.300d.txt word2vec.6B.50d.txt
```

Again, you should start seeing output pretty much immediately. It takes about 10 minutes to load and cluster the vectors, regenerate the features, and retrain the model. Please note that **all output files will be over-written** with each run.

```
(env) [maxent-ner-tagger]$ python scripts/name_tagger.py
Loading word2vec file: data/word2vec/word2vec.6B.100d.txt

Computing k=1000 clusters...
/env/lib/python3.6/site-packages/sklearn/cluster/k_means_.py:1418: RuntimeWarning: init_size=300 should be larger than k=1000. Setting it to 3*k
  init_size=init_size)

Generating train features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000
Lines processed:    60000
Lines processed:    70000
Lines processed:    80000
Lines processed:    90000
Lines processed:   100000
Lines processed:   110000
Lines processed:   120000
Lines processed:   130000
Lines processed:   140000
Lines processed:   150000
Lines processed:   160000
Lines processed:   170000
Lines processed:   180000
Lines processed:   190000
Lines processed:   200000
Lines processed:   210000

Writing output file: CoNLL/CONLL_train.pos-chunk-name.feat.csv

Generating dev features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000

Writing output file: CoNLL/CONLL_dev.pos-chunk-name.feat.csv

Generating test features...
Lines processed:        0
Lines processed:    10000
Lines processed:    20000
Lines processed:    30000
Lines processed:    40000
Lines processed:    50000

Writing output file: CoNLL/CONLL_test.pos-chunk-name.feat.csv

Training model...
X (219554, 453472)
y (219554,)

Tagging dev...
X (55044, 453472)
y (55044,)

Writing output file: output/dev.name

Tagging test...
X (50350, 453472)
y (50350,)

Writing output file: output/test.name

Scoring development set...
50822 out of 51578 tags correct
  accuracy: 98.53
5917 groups in key
6045 groups in response
5397 correct groups
  precision: 89.28
  recall:    91.21
  F1:        90.24
python scripts/name_tagger.py  733.91s user 22.76s system 101% cpu 12:22.47 total
```

### Results

As before, I tested the effect of adding the word vectors on the development set. For all experiments, I kept the best-performing set of features as determined above:

- `is_title`
- `orth_`, `lemma_`, `lower_`, `norm_`
- `shape_`
- `prefix_`, `suffix_`

I addition, I first tried word vectors of varying dimensionality (`50d`, `100d`, `200d`, `300d`), with and without including word vectors for context tokens.

| Feature set                                                            |                            Accuracy |
|:-----------------------------------------------------------------------|------------------------------------:|
| CoNLL + best-performing token features + context                       | P: 85.91<br> R: 87.21<br> **F1: 86.56** |
| CoNLL + best-performing token features + context + `50d`               | P: 87.97<br> R: 90.11<br> **F1: 89.03** |
| **CoNLL + best-performing token features + context + `100d`**          | P: 88.16<br> R: 90.50<br> **F1: 89.32** |
| CoNLL + best-performing token features + context + `200d`              | P: 87.69<br> R: 90.27<br> **F1: 88.96** |
| CoNLL + best-performing token features + context + `300d`              | P: 87.90<br> R: 90.23<br> **F1: 89.05** |
| CoNLL + best-performing token features + context + `100d` + context    | P: 86.41<br> R: 88.41<br> **F1: 87.39** |

On the development set, F1 scores improved by 2 to 3 points over the baseline model when using 100-dimensional GloVe embeddings. Adding word vector features for context tokens as well worsened the results. The best model using CoNLL features, best-performing token features, context features, and token word embeddings resulted in a **F1 score of 89.32** on the development set.

#### Binarization and clustering of embeddings

As mentioned before, I also implemented and tested binarization and clustering of embeddings. The binarization results were significantly worse than just using word vectors as features directly. Clustering on the other hand improved the results even further, especially when also including the cluster IDs of context tokens as features for each current token. Again, please refer to the [`log.md`](https://github.com/melanietosik/maxent-ner-tagger/blob/master/log.md) file for additional results and experiments.

| Feature set                                                                                 |                                Accuracy |
|:--------------------------------------------------------------------------------------------|----------------------------------------:|
| CoNLL + best-performing token features + context + `100d`                                   | P: 88.16<br> R: 90.50<br> **F1: 89.32** |
| CoNLL + best-performing token features + context + `100d` + binarization                    | P: 84.93<br> R: 88.03<br> **F1: 86.46** |
| CoNLL + best-performing token features + context + `50d` + binarization                     | P: 86.10<br> R: 88.39<br> **F1: 87.23** |
| CoNLL + best-performing token features + context + kmeans `100d; k=1000`                    | P: 88.79<br> R: 90.05<br> **F1: 89.41** |
| **CoNLL + best-performing token features + context + kmeans `100d; k=1000` + context**      | P: 89.28<br> R: 91.21<br> **F1: 90.24** |

By using the original best-performing token features as well as `k=1000` clusters of the 100-dimensional GloVe input vectors combined with context features, I was able to achieve a final F1 score of **90.24** on the development set.

