---
layout: distill
title: Impact of Co-occurrence on Factual Knowledge of Large Language Models
description:
img: assets/img/publication_preview/impact_of_cooccurrence.png
importance: 1
category: fun
related_publications: kang2023impact

authors:
  - name: Cheongwoong Kang
    url: "https://cheongwoong.github.io/"
    affiliations:
      name: KAIST
  - name: Jaesik Choi
    url: "http://sailab.kaist.ac.kr/members/jaesik/"
    affiliations:
      name: KAIST

bibliography: custom.bib
date: 2023-10-31
---

This is the project page for the paper "[Impact of Co-occurrence on Factual Knowledge of Large Language Models](https://aclanthology.org/2023.findings-emnlp.518.pdf)". You may refer to the additional materials, including the [code](https://github.com/CheongWoong/impact_of_cooccurrence), [poster](/assets/pdf/impact_of_cooccurrence/poster.pdf) and [slides](/assets/pdf/impact_of_cooccurrence/slides.pdf).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/publication_preview/impact_of_cooccurrence.png" title="overall framework" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    This figure shows an overall framework of our correlation analysis between co-occurrence counts and factual knowledge of LLMs. We assume that if the target model heavily relies on subject-object co-occurrence, it is more likely to recall the most co-occurring word without accurate semantic understanding. For instance, in this hypothetical example, the model <b>fails to answer the question about the capital of Canada by generating the most frequently co-occurring word 'Toronto'</b>, while the correct answer is 'Ottawa'. This indicates that relying heavily on co-occurrence statistics may have potential errors.
</div>

## Abstract
Large language models (LLMs) often make factually incorrect responses despite their success in various applications. In this paper, we hypothesize that relying heavily on simple co-occurrence statistics of the pre-training corpora is one of the main factors that cause factual errors. Our results reveal that LLMs are vulnerable to the **co-occurrence bias, defined as preferring frequently co-occurred words over the correct answer**. Consequently, LLMs struggle to recall facts whose subject and object rarely co-occur in the pre-training dataset although they are seen during finetuning. We show that co-occurrence bias remains despite scaling up model sizes or finetuning. Therefore, we suggest finetuning on a debiased dataset to mitigate the bias by filtering out biased samples whose subject-object co-occurrence count is high. Although debiased finetuning allows LLMs to memorize rare facts in the training set, it is not effective in recalling rare facts unseen during finetuning. Further research in mitigation will help build reliable language models by preventing potential errors. The code is available at <https://github.com/CheongWoong/impact_of_cooccurrence>.

## Factual Knowledge Probing
**The LAMA Probe**

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/factual_knowledge_probing_procedure.png" title="factual knowledge probing procedure" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We adopt the LAMA-TREx dataset <d-cite key="elsahar2018t,petroni2019language"></d-cite>, which consists of 41 relations, to probe factual knowledge of LLMs. Facts are represented as subject-relation-object triples (e.g. 'Canada'-'capital'-'Ottawa'). Each fact is converted to a natural language form based on a pre-defined set of templates for relations (e.g. "The capital of Canada is Ottawa."). Then, it is converted to a Cloze statement by masking an object (e.g. "The capital of Canada is \[MASK\]"). To query unidirectional LMs, we use a sentence truncated right before the mask token (e.g. "The capital of Canada is").

**Metrics**

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/factual_knowledge_probing_metrics.png" title="factual knowledge probing metrics" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Following the knowledge base completion literature <d-cite key="bordes2011learning,bordes2013translating"></d-cite>, we use **hits@1** to evaluate the performance on factual knowledge probing. Hits@1 is 1 if the correct answer is ranked in the top 1 prediction, otherwise 0. When computing hits@1, we remove other valid objects for a subject-relation pair other than the one we test.

**Restricted Candidate Sets**

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/restricted_candidate_sets_remove_stopwords.png" title="restricted candidate sets remove stopwords" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    An example of restricting candidate sets (<i>remove stopwords</i>).
</div>

Since **LLMs are not trained to act as knowledge bases**, we use **restricted output candidate sets** following the recent work <d-cite key="xiong2020pretrained,ravichander2020systematicity,kassner2021multilingual,elazar2021measuring"></d-cite>. Specifically, we use three different settings to restrict output vocabularies to study whether LLMs are capable of recognizing appropriate object candidates or not: (1) *remove stopwords*, (2) *gold objects* and (3) *gold objects (relation-wise)*. The *remove stopwords* simply removes stopwords in the stopword list of NLTK <d-cite key="bird2009natural"></d-cite> from the output candidates. The *gold objects* restricts the output vocabulary to the set of gold objects in the whole dataset. Similarly, the *gold objects (relation-wise)* restricts the output candidates to the set of gold objects for each relation in the dataset.

## Factual Knowledge Probing with Co-occurrence Statistics
**Co-occurrence Statistics**

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/cooccurrence_counting_pipeline.png" title="cooccurrence counting pipeline" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

We consider the **co-occurrence statistics of the pre-training dataset** of target models. Specifically, we focus on **subject-object co-occurrence**, motivated by the concept of distant supervision, which shows that a sentence often contains the triple if it contains a subject and an object of a triple <d-cite key="mintz2009distant"></d-cite>. Due to computational burdens from the large amount of documents, we **count whether an entity pair appears in the same document or not**, instead of using a sliding window approach.

**Correlation Metrics**

To analyze the correlation between factual knowledge of LLMs and co-occurrence statistics, we **plot hits@1 of the target LLMs against the conditional probability** of the gold object given a subject. Here, we use the co-occurrence statistics of the pre-training corpora to compute conditional probabilities. We divide the samples into multiple frequency (conditional probability) bins and report the average hits@1 for each bin.

## Experimental Setup
We test open-source versions of GPT-3 <d-cite key="brown2020language"></d-cite> with four different model sizes: **GPT-Neo 125M, GPT-Neo 1.3B, GPT-Neo 2.7B, and GPT-J 6B** <d-cite key="gpt-neo,gpt-j"></d-cite>, which are publicly available on Huggingface's transformers <d-cite key="wolf-etal-2020transformers"></d-cite>. These models are pre-trained on **the Pile** <d-cite key="gao2020pile"></d-cite>, which is a publicly available dataset that consists of 800GB of high-quality texts from 22 different sources.

## Results
**Factual Knowledge Probing**

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/hits_1_zeroshot_test.jpg" title="hits_1_zeroshot_test" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/hits_1_finetuned_test.jpg" title="hits_1_finetuned_test" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    <b>Effects of model sizes and restricted candidate sets:</b> We plot micro-average hits@1 on the test set. <b>(left)</b> In the zero-shot setting, we observe that <b>hits@1 is higher as the model is larger and as the output vocabulary is restricted to a smaller set</b>. <b>(right)</b> In the finetuned setting, we observe that the effect of model sizes and restricted candidate sets is marginal. <b>Effects of finetuning:</b> <b>(left->right)</b> We observe that <b>finetuning boosts the overall performance</b>.
</div>

**Impact of Co-occurrence**

<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/condprob_correlation_zeroshot_remove_stopwords_test.jpg" title="condprob_correlation_zeroshot_remove_stopwords_test" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/condprob_correlation_finetuned_remove_stopwords_test.jpg" title="condprob_correlation_finetuned_remove_stopwords_test" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    <b>The correlation between co-occurrence statistics and factual knowledge probing accuracy:</b> We plot hits@1 against $P_{pretrain}(obj|subj)$, the conditional probability of the gold object given a subject, on the test set in the <i>remove stopwords</i> setting. In both (left) zero-shot and (right) finetuned settings, we observe a strong correlation: <b>hits@1 is lower as the co-occurrence count is lower</b>. As a result, <b>LLMs struggle to recall rare facts</b>. We observe that such <b>correlation remains despite scaling up model sizes or finetuning</b>.
</div>
<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/openai_api_condprob_correlation_zeroshot_remove_stopwords_test.jpg" title="openai_api_condprob_correlation_zeroshot_remove_stopwords_test" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    <b>Correlational analysis of larger models:</b> We test <b>larger models (GPT-3.5 175B and ChatGPT)</b> on the subset of test data in the <i>remove stopwords</i> setting, verifying that correlation remains despite scaling up model sizes.
</div>
<div class="row justify-content-sm-center">
    <div class="col-sm-6 mt-3 mt-md-0">
        {% include figure.html path="assets/img/impact_of_cooccurrence/count_biased_cases.png" title="count biased cases" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption_left">
    The quantitative failure analysis of GPT-J 6B, <b>counting biased cases, in which the correct answer is overridden by a word with higher co-occurrence</b>. The ratio of biased cases to the total failure cases is reported. We observe that a word with higher co-occurrence is preferred over the correct answer in a total of 38% of the failure cases. The results of different frequency bins show that the <b>co-occurrence bias is more problematic when recalling rare facts</b>.
</div>

## Conclusion

- LLMs are vulnerable to the **co-occurrence bias, defined as preferring frequently co-occurred words over the correct answer**.
- Consequently, **LLMs struggle to recall facts whose subject and object rarely co-occur** in the pre-training dataset.
- Co-occurrence bias remains **despite scaling up model sizes or finetuning**.
- Therefore, we **suggest further investigation on mitigating co-occurrence bias to ensure the reliability of language models** by preventing potential harms.
