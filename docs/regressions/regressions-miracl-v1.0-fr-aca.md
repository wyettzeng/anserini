# Anserini Regressions: MIRACL (v1.0) &mdash; Arabic

**Models**: bag-of-words approaches using `AutoCompositeAnalyzer`

This page documents BM25 regression experiments for [MIRACL (v1.0) &mdash; Arabic](https://github.com/project-miracl/miracl) using `AutoCompositeAnalyzer`.

The exact configurations for these regressions are stored in [this YAML file](../../src/main/resources/regression/miracl-v1.0-fr-aca.yaml).
Note that this page is automatically generated from [this template](../../src/main/resources/docgen/templates/miracl-v1.0-fr-aca.template) as part of Anserini's regression pipeline, so do not modify this page directly; modify the template instead.

From one of our Waterloo servers (e.g., `orca`), the following command will perform the complete regression, end to end:

```
python src/main/python/run_regression.py --index --verify --search --regression miracl-v1.0-fr-aca
```

## Indexing

Typical indexing command:

```
target/appassembler/bin/IndexCollection \
  -collection MrTyDiCollection \
  -input /path/to/miracl-v1.0-fr \
  -index indexes/lucene-index.miracl-v1.0-fr-aca/ \
  -generator DefaultLuceneDocumentGenerator \
  -threads 1 -storePositions -storeDocvectors -storeRaw -language fr -useAutoCompositeAnalyzer \
  >& logs/log.miracl-v1.0-fr &
```

See [this page](https://github.com/project-miracl/miracl) for more details about the MIRACL corpus.
For additional details, see explanation of [common indexing options](../../docs/common-indexing-options.md).

## Retrieval

After indexing has completed, you should be able to perform retrieval as follows:

```
target/appassembler/bin/SearchCollection \
  -index indexes/lucene-index.miracl-v1.0-fr-aca/ \
  -topics tools/topics-and-qrels/topics.miracl-v1.0-fr-dev.tsv \
  -topicreader TsvString \
  -output runs/run.miracl-v1.0-fr.bm25.topics.miracl-v1.0-fr-dev.txt \
  -bm25 -hits 100 -language fr -useAutoCompositeAnalyzer &
```

Evaluation can be performed using `trec_eval`:

```
tools/eval/trec_eval.9.0.4/trec_eval -c -m ndcg_cut.10 tools/topics-and-qrels/qrels.miracl-v1.0-fr-dev.tsv runs/run.miracl-v1.0-fr.bm25.topics.miracl-v1.0-fr-dev.txt
tools/eval/trec_eval.9.0.4/trec_eval -c -m recall.100 tools/topics-and-qrels/qrels.miracl-v1.0-fr-dev.tsv runs/run.miracl-v1.0-fr.bm25.topics.miracl-v1.0-fr-dev.txt
```

## Effectiveness

With the above commands, you should be able to reproduce the following results:

| **nDCG@10**                                                                                                  | **BM25**  |
|:-------------------------------------------------------------------------------------------------------------|-----------|
| [MIRACL (French): dev](https://github.com/project-miracl/miracl)                                             | 0.1832    |
| **R@100**                                                                                                    | **BM25**  |
| [MIRACL (French): dev](https://github.com/project-miracl/miracl)                                             | 0.6528    |
