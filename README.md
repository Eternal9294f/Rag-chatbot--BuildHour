# RAG based:  HDFC Mutual Fund Assistant

A facts-only mutual fund chatbot built to explore how **Retrieval-Augmented Generation (RAG)** can make financial research faster while keeping answers grounded in source information, with five official HDFC Mutual Fund scheme pages (Large Cap, Flexi Cap, ELSS Tax Saver, Small Cap, Balanced Advantage). It answers only from those pages, keeps every answer to ≤3 sentences, shows exactly one source link, and appends "Last updated from sources: ". Opinion, advice, return-comparison, and PII-bearing questions are refused before any retrieval. 

**Live:** 

---

## Why I built this

This project was built as part of **NextLeap's Product Management Fellowship, Milestone 4**.

I wanted to explore a simple product problem:

**How can AI make financial information easier to research without becoming a source of financial advice?**

Mutual fund information is publicly available, but finding a specific fact often means searching through long pages and documents.

A chatbot makes the interaction much simpler.

Instead of searching manually, a user can ask:

> "What is the expense ratio of this fund?"

and get an answer in seconds.

But speed alone isn't enough.

When the subject is money, a confident but unsupported answer can be much worse than no answer at all.

So I built the product around a different principle:

**Retrieve the evidence first. Then answer from it.**

---

## What I built

The chatbot answers factual questions about a selected set of mutual funds.

It covers information such as:

* Expense ratio
* Exit load
* Minimum SIP
* Risk level
* Benchmark
* Fund managers
* Lock-in period
* AUM
* Other factual information available in the source material

The experience is intentionally **facts-only**.

It does not provide:

* Investment recommendations
* Personalised financial advice
* Return predictions
* "Which fund is better?" decisions
* Live market information
* Information that isn't supported by its available sources

Answers include the relevant source information and the date associated with the underlying data.

---

## How RAG works here

The core product uses **Retrieval-Augmented Generation** rather than relying entirely on the LLM's general knowledge.

The pipeline is:

**Source documents**
↓
**Clean & split into smaller chunks**
↓
**Generate embeddings**
↓
**Store in vector database**
↓
**User asks a question**
↓
**Retrieve semantically relevant chunks**
↓
**Check retrieval confidence**
↓
**Generate answer from retrieved context**
↓
**Return answer + source**

The LLM isn't treated as the source of truth.

The retrieved information is.

This gives the product a controlled knowledge boundary instead of asking the model to answer from everything it may already know.

---

## The knowledge base

The first version intentionally focuses on a small, controlled corpus of mutual fund information.

The source material covers **five HDFC mutual fund schemes**:

* HDFC Large Cap Fund
* HDFC Flexi Cap Fund
* HDFC ELSS Tax Saver Fund
* HDFC Small Cap Fund
* HDFC Balanced Advantage Fund

The information was collected from publicly available Groww fund pages and represents information available as of **27 September 2026**.

The source material is cleaned and divided into smaller factual chunks before being indexed.

This was a deliberate product decision.

Rather than trying to cover hundreds of funds immediately, I wanted a smaller knowledge base that could be **tested, understood, and controlled**.

---

## Retrieval

The system uses semantic search to find information relevant to the user's question.

The documents are converted into embeddings using **all-MiniLM-L6-v2** and stored in **ChromaDB**.

When a question arrives, the system retrieves the most relevant chunks rather than sending the entire knowledge base to the LLM.

There is also a **similarity threshold**.

If the retrieved information is not relevant enough to the question, the chatbot refuses instead of generating an answer from weak context.

This creates an important boundary:

**Good retrieval → Generate**

**Weak retrieval → Refuse**

The threshold is calibrated against the available corpus rather than being chosen only because it makes the demo look good.

---

## Guardrails

RAG reduces hallucination risk, but retrieval alone doesn't solve every problem.

The product therefore has guardrails before and after retrieval.

### 1. Scope and safety check

Questions asking for investment advice, opinions, return comparisons, or personal financial information are outside the product's scope.

These are refused before normal answer generation.

### 2. Retrieval gate

Even a factual question can fail if the system cannot find sufficiently relevant information.

The similarity threshold acts as a second gate.

If the retrieved context is too weak, the LLM is not called.

### 3. Grounded generation

When the system does generate an answer, the model receives the retrieved source material as context.

The answer is expected to stay within that evidence.

### 4. Number checking

Financial answers frequently contain numbers.

So generated numbers are checked against the retrieved source information rather than simply trusting the model's output.

This is particularly important for things such as:

* Expense ratios
* SIP amounts
* Exit loads
* AUM
* Lock-in periods

The principle is simple:

**The model can explain the information. It shouldn't invent the information.**

---

## Conversation context

Users rarely repeat their entire question.

For example:

**"What is the expense ratio of HDFC Small Cap Fund?"**

followed by:

**"What about the minimum SIP?"**

The second question depends on the context of the first.

The chatbot therefore uses conversation history when resolving follow-up questions.

The retrieval layer can use a wider history window to identify the correct referent, while the final generation step receives a smaller context window.

This keeps follow-up questions natural without unnecessarily increasing the amount of conversation sent to the model.

---

## Source transparency

One of the core product ideas is that the user shouldn't have to blindly trust the chatbot.

Each factual answer provides:

* The answer
* The relevant source
* The source update information

The source information is handled separately from the generated response so that the chatbot isn't responsible for inventing its own citation.

The intended experience is:

**"Here's the answer."**

**"Here's where it came from."**

**"Here's when that information was last updated."**

---

## Privacy and data boundaries

The product is deliberately limited in what information it handles.

Personal financial information such as PAN, Aadhaar, OTPs, or account details is outside the intended use case and is refused.

The system does not need personal financial data to answer its core questions.

The RAG pipeline also limits what reaches the LLM by sending retrieved chunks rather than the entire source corpus.

This reinforces another product principle:

**Only send the information that is necessary to answer the question.**

---

## Evaluation

I didn't want to evaluate the chatbot only by asking questions that were easy to answer.

I created a test set of **128 questions**, including both normal and deliberately difficult cases.

The evaluation included:

* Factual fund questions
* Different ways of asking the same question
* Follow-up questions
* Hinglish queries
* Out-of-scope questions
* Investment advice requests
* Personal information
* Gibberish
* Unrelated questions
* Requests for returns
* Requests for live information

The goal was to evaluate both sides of the product:

**Can it answer when it has evidence?**

and

**Can it refuse when it doesn't?**

The current evaluation achieved:

**128 / 128 tests passed.**

That made refusal behaviour an important success criterion rather than treating every refusal as a failure.

---

## Product decisions

A few decisions shaped the first version.

### Start narrow

Five funds are easier to control and evaluate than an enormous corpus.

The initial goal was reliability, not coverage.

### Facts over recommendations

The product helps users research information without making the decision for them.

### Refusal is a feature

If the system doesn't have enough evidence, saying **"I don't know"** is preferable to producing a plausible answer.

### No live returns

Returns and live market information introduce a different data and product problem.

They were intentionally left outside the first version.

### No unnecessary features

The first version focuses on the core loop:

**Ask → Retrieve → Answer → Verify**

Features such as accounts, saved history, and personalised recommendations were not necessary to test that core experience.

---

## What I learned

The biggest lesson was that **RAG is not simply an LLM connected to a vector database.**

The technical architecture is only part of the product.

The harder questions were:

**What belongs in the knowledge base?**

**How should information be chunked?**

**How much retrieval is enough?**

**When should the model be called?**

**When should the product refuse?**

**How do you evaluate an answer that sounds correct?**

I also learned that AI products can fail quietly.

A chatbot can give a polished answer that looks completely reasonable while being unsupported by its source.

That means evaluation isn't something to add at the end.

**Testing is part of the product.**

---

## What I'd improve

The current version is intentionally limited.

If I continued building it, I'd focus on:

### Better sources

Move toward official AMC and other authoritative sources wherever possible.

### Automatic updates

Build a process to detect changes in source documents and automatically refresh the knowledge base.

### Better retrieval

Experiment with chunking, embeddings, reranking, and retrieval strategies to improve context quality.

### Real-user evaluation

Move beyond a controlled test set and collect questions from actual users.

### Better product metrics

Track metrics such as:

* Answer success rate
* Correct refusal rate
* Unsupported answer rate
* Source accuracy
* Follow-up success rate
* User-reported errors
* Response time

The next stage would be less about making the chatbot answer more questions and more about understanding:

**Which answers do users actually trust and find useful?**

---

## Why this project matters to me

This project helped me understand the difference between **building an AI feature** and **building an AI product**.

The model is only one part.

The actual product is the system around it:

**Problem → Data → Retrieval → Guardrails → Experience → Evaluation → Trust**

That's what made this project particularly valuable to me as a Product Management project.

I wasn't only trying to answer:

**"Can I build a RAG chatbot?"**

I was trying to understand:

**"What should this product know, what should it refuse to know, and how do I prove that it is working?"**

---

## Tech

`Python` · `FastAPI` · `RAG` · `ChromaDB` · `all-MiniLM-L6-v2` · `Embeddings` · `Groq` · `HTML/CSS/JavaScript` · `Docker` · `Render`

Built as part of **NextLeap Product Management Fellowship, Milestone 4**, exploring RAG, AI product design, information retrieval, guardrails, evaluation, and trustworthy AI experiences.
