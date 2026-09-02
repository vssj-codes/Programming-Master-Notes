# Table-of-Contents

<!-- toc -->

- [Full-Text Search and Elasticsearch](#full-text-search-and-elasticsearch)
  * [The Problem — Traditional Database Search](#the-problem--traditional-database-search)
  * [The Librarian Analogy](#the-librarian-analogy)
  * [The Solution — Inverted Index](#the-solution--inverted-index)
    + [Example](#example)
  * [How Elasticsearch Works](#how-elasticsearch-works)
    + [Architecture](#architecture)
    + [Relevance Scoring — BM25 Algorithm](#relevance-scoring--bm25-algorithm)
  * [Key Features of Full-Text Search](#key-features-of-full-text-search)
    + [Typo Tolerance (Fuzzy Matching)](#typo-tolerance-fuzzy-matching)
    + [Type-Ahead / Auto-Complete](#type-ahead--auto-complete)
    + [Relevance-Based Ordering](#relevance-based-ordering)
  * [Elasticsearch vs PostgreSQL Full-Text Search](#elasticsearch-vs-postgresql-full-text-search)
    + [When to use which](#when-to-use-which)
  * [ELK Stack](#elk-stack)
  * [Practical Advice](#practical-advice)

<!-- tocstop -->

---

# Full-Text Search and Elasticsearch

**Source:** Sriniously — Backend from First Principles (Video 15)
**Link:** [Watch](https://www.youtube.com/watch?v=estH64OkwxU)

---

## The Problem — Traditional Database Search

A typical relational database search query:

```sql
SELECT * FROM products
WHERE name ILIKE '%laptop%'
   OR description ILIKE '%laptop%';
```

- `ILIKE` = case-insensitive pattern match
- `%` = match any characters before/after the term

**Works fine at small scale (5,000 rows).** As the dataset grows to millions of rows, this same query degrades from ~50ms to 30+ seconds.

**Two fundamental problems:**
1. **Speed** — the database scans every single row and performs character-by-character pattern matching
2. **No relevance** — results are returned in arbitrary order; a product where "laptop" appears once in a description ranks the same as one named "MacBook Pro Laptop"

---

## The Librarian Analogy

Think of a relational database as a librarian with a fatal flaw:

- To find books about "machine learning", the librarian reads **every book title and content** in the entire library, one by one
- Thorough — will find all matches
- Painfully slow at scale (millions of books = millions of rows)
- **No sense of relevance** — returns results in arbitrary order; a book that mentions "machine learning" once on the last page ranks the same as a book titled "Introduction to Machine Learning"

---

## The Solution — Inverted Index

The revolutionary idea: **flip the problem**.

Instead of scanning books to find terms → **scan terms to find books**.

**Traditional approach:** Document → search for term
**Inverted index:** Term → list of documents where it appears + positions

### Example

| Term | Documents (book → pages) |
|------|--------------------------|
| `machine` | Introduction to Machine Learning (p.1, 15, 23), The Machine Age (p.5, 89), Coffee Machine Manual (p.1) |
| `learning` | Introduction to Machine Learning (p.1, 16, 24), Learning to Cook (p.3, 7), Deep Learning Fundamentals (p.2, 8, 14) |

When a user searches "machine learning":
1. Look up `machine` in the index → instantly get all books
2. Look up `learning` in the index → instantly get all books
3. Score and rank the results by relevance

No full-table scan. Lookup is near-instant regardless of dataset size.

> **This is why it's called "inverted" index** — traditional index goes document → terms; inverted index goes term → documents.

---

## How Elasticsearch Works

**Elasticsearch** is a search engine built on top of **Apache Lucene** — the core library that implements the inverted index.

> Apache Lucene is not unique to Elasticsearch. PostgreSQL's full-text search and many other tools also use it.

### Architecture

- Data in Elasticsearch is stored as **JSON documents** (similar to MongoDB)
- Documents are organized into **indices** (like database tables)
- When a document is indexed, Elasticsearch builds an inverted index of all text fields

### Relevance Scoring — BM25 Algorithm

Elasticsearch uses the **BM25** algorithm to rank results. Four key factors:

| Factor | What it measures |
|--------|-----------------|
| **Term Frequency** | How often the search term appears in a single document |
| **Document Frequency** | How common the term is across all documents (rare terms = more relevant) |
| **Document Length** | Shorter documents with the term score higher than longer ones |
| **Field Boosting** | Where the term appears — title > description > content |

**Field boosting example:** If "laptop" appears in the product title, that result ranks higher than one where "laptop" only appears in the description.

> Field boosting is configurable — you can define your own scoring priorities in the Elasticsearch query DSL.

---

## Key Features of Full-Text Search

### Typo Tolerance (Fuzzy Matching)

Search for `treading` → returns results for `trending`.

Elasticsearch derives intent from context and corrects common typos automatically, so user typos don't break search results.

### Type-Ahead / Auto-Complete

As the user types, Elasticsearch returns instant suggestions — powers the search-as-you-type interfaces seen on Amazon, Google, etc.

### Relevance-Based Ordering

Results are ranked by relevance score, not insertion order. The most meaningful match comes first.

---

## Elasticsearch vs PostgreSQL Full-Text Search

Both can handle full-text search. PostgreSQL has built-in full-text search support.

**Benchmark (50,000 product reviews):**

| Query | PostgreSQL (`ILIKE`) | Elasticsearch |
|-------|---------------------|---------------|
| "laptop" | ~3–4 seconds | ~1 second |
| "something" | ~7.5 seconds | ~500ms |

**Same number of results, dramatically different latency.**

### When to use which

| Option | Use when |
|--------|---------|
| **PostgreSQL full-text search** | Already using PostgreSQL, moderate search requirements, simple use case |
| **Elasticsearch** | Company already has it (e.g. for ELK stack), large scale, advanced relevance tuning, typo tolerance |

---

## ELK Stack

Elasticsearch is also a core part of the **ELK Stack** — a popular log management solution:

| Letter | Technology | Role |
|--------|-----------|------|
| **E** | Elasticsearch | Store and search logs at high speed |
| **L** | Logstash | Collect, transform, ship logs |
| **K** | Kibana | Visualize and explore log data |

If your company already runs Elasticsearch for log management, extending it for full-text search makes sense — no extra infrastructure.

---

## Practical Advice

- **Databases** — master thoroughly; involved in ~99% of backend code
- **Elasticsearch** — know when to reach for it; the docs and snippets cover most use cases without deep internals knowledge

**When you need full-text search:**
1. Check if PostgreSQL's built-in full-text search is sufficient
2. If you need more scale, relevance tuning, typo tolerance, or already have Elasticsearch → use Elasticsearch

You don't need to master BM25 internals or Lucene architecture to be productive. Know the use case, use the docs, implement the feature.
