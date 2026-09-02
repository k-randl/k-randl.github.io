Imagine you ask ChatGPT, Claude, Gemini, or Perplexity a medical question and it responds with an answer plus a list of sources. Internally, it just retrieved a list of online documents, ranked them by relevancy, and then generated an answer with these documents injected into the prompt. But what if it mostly relied on reddit comments while barely paying attention to proper medical information?

From the outside, you'd never know.

This is a question that's surprisingly hard to answer: when a RAG system gives you an answer, is it actually *using* the documents? Or is it quietly ignoring the top-ranked document and building its answer around something buried further down the list, or worse, making up content?

A new framework called **RAG-E**, developed by researchers at Stockholm University, TU Berlin, and the Athens University of Economics and Business, is designed to answer exactly that question. The work was accepted to **EMNLP Findings 2026** [\[2026.emnlp.org\]](https://2026.emnlp.org/) and will be presented at the conference later this year. The preprint is available on arXiv [\[arxiv.org\]](https://arxiv.org/abs/2601.21803).

## The missing piece in RAG evaluation

A standard RAG pipeline has two main components:

1. First, **the Retriever** searches a knowledge base (e.g. a list of websites) and ranks documents by relevance to the query.  
2. Then, **the Generator**, a large language model, reads those documents and writes an answer.

Most existing explainability tools look at one of these components in isolation. Some tools explain *why* the retriever picked certain documents. Others explain *which* documents the generator leaned on while writing its answer. What's been missing is a way to compare the two: to ask whether the documents the retriever thought were important are the same ones that the generator actually relied on.

That gap matters. If important documents are routinely ignored, retrieval effort is being wasted. More importantly, users lose confidence that answers are grounded in your strongest evidence rather than a less relevant source that happened to catch the model's attention.

## ![RAG-E overview](https://k-randl.github.io/img/posts/2026-09-02-image1.png)

RAG-E overview. We detect important spans influencing both the retrieval and generation steps. **(a):** Visual example of RAG-E’s explanations (generated using Arctic Embed 2 and Llama). **(b):** Explanations are based on integrated gradients for the retriever and SHAP for the generator.

## What RAG-E provides

RAG-E is an end-to-end auditing framework designed to reveal how retrieval and generation interact. It has two main components:

**1\. Better explanations for the entire RAG pipeline.** RAG-E explains both why documents were retrieved and how much they influenced the final answer, by adapting and refining the state-of-the-art methods *Integrated Gradients* (for retrieval) and *SHAP* (for generation).

**2\. WARG: a single number for retriever-generator alignment.** The framework's headline contribution is *WARG* (Weighted Alignment between Retriever and Generator), a metric that compares document importance during retrieval with document influence during generation and summarizes the relationship in a single score between \-1 and 1\. High WARG indicates that the model is relying on the documents judged most relevant. Low or negative WARG suggests a mismatch.

The researchers show that WARG captures this relationship more effectively than traditional measures such as Pearson or Spearman correlation, largely because document usage in generation tends to be close to binary: documents are often either clearly used or largely ignored.

## What happens in real systems?

Evaluating RAG-E across three QA datasets (PopQA, QAMPARI, and TREC CAsT), two retriever models, and three open-weight LLMs, the study finds that misalignment is far from rare. Looking just at the top three retrieved documents, the generator ignored the retriever's top pick anywhere from 16% to nearly 78% of the time, depending on the model combination and dataset. In other words, in a lot of everyday RAG setups, a meaningful chunk of retrieval effort is going to waste.

The good news: when WARG is high, meaning the retriever and generator agree, the generated answer is substantially more likely to actually contain the correct, ground-truth information (more than twice as likely on PopQA, and over four times as likely on QAMPARI). So this isn't just an academic curiosity; alignment tracks with getting the answer right.

## How you can use it

RAG-E isn't just a paper; it's a working tool. The authors released a Python package (`pip install rag-exp`), with source code available on GitHub [\[github.com\]](https://github.com/k-randl/Interpretable_RAG). In practice, you can point it at your own retriever and generator, feed it a query and its documents, and get back:

- token-level saliency maps showing which words in the query and documents drove retrieval,  
- a breakdown of how much each retrieved document influenced the final generated answer,  
- and a WARG score summarizing how well the two components agree.

That makes it useful both as a diagnostic tool, spotting when your RAG pipeline is quietly wasting retrieval budget or getting distracted by irrelevant context, and, longer-term, as a potential training or optimization signal for building RAG systems where retrieval and generation are actually working together instead of past each other.

As RAG systems increasingly show up in high-stakes settings like healthcare and law, that kind of visibility isn't just nice to have. It's the difference between a system you can audit and one you're taking on faith.