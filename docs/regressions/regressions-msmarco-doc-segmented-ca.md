# Anserini Regressions: TREC 2020 Deep Learning Track (Document)

**Models**: various bag-of-words approaches on segmented documents

This page describes experiments, integrated into Anserini's regression testing framework, on the [TREC 2020 Deep Learning Track document ranking task](https://trec.nist.gov/data/deep2020.html).

Note that the NIST relevance judgments provide far more relevant documents per topic, unlike the "sparse" judgments provided by Microsoft (these are sometimes called "dense" judgments to emphasize this contrast).
For additional instructions on working with MS MARCO document collection, refer to [this page](../../docs/experiments-msmarco-doc.md).

+ **Indexing Condition:** each MS MARCO document is first segmented into passages, each passage is treated as a unit of indexing
+ **Expansion Condition:** none

In the passage (i.e., segment) indexing condition, we select the score of the highest-scoring passage from a document as the score for that document to produce a document ranking; this is known as the MaxP technique.

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/msmarco-doc-segmented-ca.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/msmarco-doc-segmented-ca.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

Note that in November 2021 we discovered issues in our regression tests, documented [here](../../docs/experiments-msmarco-doc-doc2query-details.md).
As a result, we have had to rebuild all our regressions from the raw corpus.
These new versions yield end-to-end scores that are slightly different, so if numbers reported in a paper do not exactly match the numbers here, this may be the reason.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression msmarco-doc-segmented-ca
```

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection JsonCollection \
  -input /path/to/msmarco-doc-segmented \
  -index indexes/lucene-index.msmarco-doc-segmented-ca/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 16 -storePositions -storeDocvectors -storeRaw -analyzeWithHuggingFaceTokenizer bert-base-uncased -useCompositeAnalyzer \
  >& logs/log.msmarco-doc-segmented &
```

The directory `/path/to/msmarco-doc-segmented/` should be a directory containing the segmented corpus in Anserini's jsonl format.
See [this page](../../docs/experiments-msmarco-doc-doc2query-details.md) for how to prepare the corpus.

For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

Topics and qrels are stored [here](https://github.com/castorini/anserini-tools/tree/master/topics-and-qrels), which is linked to the Anserini repo as a submodule.
The regression experiments here evaluate on the 5193 dev set questions.

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.msmarco-doc-segmented-ca/ \
  -topics tools/topics-and-qrels/topics.msmarco-doc.dev.txt \
  -topicreader TsvInt \
  -output runs/run.msmarco-doc-segmented.bm25-default.topics.msmarco-doc.dev.txt \
  -bm25 -hits 10000 -selectMaxPassage -selectMaxPassage.delimiter "#" -selectMaxPassage.hits 1000 -analyzeWithHuggingFaceTokenizer bert-base-uncased -useCompositeAnalyzer &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -c -m map tools/topics-and-qrels/qrels.msmarco-doc.dev.txt runs/run.msmarco-doc-segmented.bm25-default.topics.msmarco-doc.dev.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -M 100 -m recip_rank tools/topics-and-qrels/qrels.msmarco-doc.dev.txt runs/run.msmarco-doc-segmented.bm25-default.topics.msmarco-doc.dev.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.msmarco-doc.dev.txt runs/run.msmarco-doc-segmented.bm25-default.topics.msmarco-doc.dev.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.1000 tools/topics-and-qrels/qrels.msmarco-doc.dev.txt runs/run.msmarco-doc-segmented.bm25-default.topics.msmarco-doc.dev.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **AP@1000**                                                                                                  | **BM25 (default)**|
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [MS MARCO Doc: Dev](https://github.com/microsoft/MSMARCO-Document-Ranking)                                   | 0.2787    |
| **RR@100**                                                                                                   | **BM25 (default)**|
| [MS MARCO Doc: Dev](https://github.com/microsoft/MSMARCO-Document-Ranking)                                   | 0.2781    |
| **R@100**                                                                                                    | **BM25 (default)**|
| [MS MARCO Doc: Dev](https://github.com/microsoft/MSMARCO-Document-Ranking)                                   | 0.7990    |
| **R@1000**                                                                                                   | **BM25 (default)**|
| [MS MARCO Doc: Dev](https://github.com/microsoft/MSMARCO-Document-Ranking)                                   | 0.9286    |

Explanation of settings:

+ The setting "default" refers the default BM25 settings of `k1=0.9`, `b=0.4`.

Note that in the official evaluation for document ranking, all runs were truncated to top-100 hits per query (whereas all top-1000 hits per query were retained for passage ranking).
Thus, average precision is computed to depth 100 (i.e., AP@100); nDCG@10 remains unaffected.
Remember that we keep qrels of _all_ relevance grades, unlike the case for passage ranking, where relevance grade 1 needs to be discarded when computing certain metrics.
Here, we retrieve 1000 hits per query, but measure AP at cutoff 100 (e.g., AP@100).
Thus, the experimental results reported here are directly comparable to the results reported in the [track overview paper](https://arxiv.org/abs/2102.07662).

Note that [#1721](https://github.com/castorini/anserini/issues/1721) slightly change the results, since we corrected underlying issues with data preparation.
