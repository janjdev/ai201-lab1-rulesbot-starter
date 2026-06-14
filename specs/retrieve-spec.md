# Spec: `retrieve()`

**File:** `retriever.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user's natural language query, find the most relevant chunks from the vector store using semantic similarity search. Return them ranked by relevance so that `generate_response()` can use them as context.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's natural language question |
| `n_results` | `int` | Maximum number of chunks to return (default: `N_RESULTS` from `config.py`) |

**Output:** `list[dict]`

Each dict in the returned list must contain exactly these keys:

| Key | Type | Description |
|-----|------|-------------|
| `"text"` | `str` | The chunk text |
| `"game"` | `str` | The game name this chunk came from |
| `"distance"` | `float` | Cosine distance score — lower means more similar to the query |

Results should be ordered from most to least relevant (lowest to highest distance). Returns an empty list `[]` if the collection contains no documents.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Query approach

*Describe how you will use `_collection.query()` to find relevant chunks. What arguments will you pass, and why?*

```
I will use _collection.query() with query_texts=[query], n_results=n_results, and include=["documents", "metadatas", "distances"]. This lets ChromaDB embed the user's question with the same embedding function used during ingestion, compare it to the stored rule chunks, and return the closest chunks by cosine distance.
```

---

### Return structure

*Sketch out what one item in your return list looks like as a concrete example. Where does each field come from in the query results?*

```
One returned item will look like:

{
  "text": "When a 7 is rolled, no one collects any resources...",
  "game": "Catan",
  "distance": 0.142
}

The "text" field comes from results["documents"][0].
The "game" field comes from results["metadatas"][0], specifically the metadata stored during ingestion.
The "distance" field comes from results["distances"][0].
```

---

### Handling the nested result structure

*`_collection.query()` returns nested lists. Describe what index you need to access to get the actual list of results for a single query, and why the nesting exists.*

```
_collection.query() returns nested lists because it can handle multiple query strings at once. Since RulesBot only sends one query at a time, the actual results are inside index [0]. So I need to use results["documents"][0], results["metadatas"][0], and results["distances"][0].
```

---

### Relevance threshold

*Will you filter out results above a certain distance score, or return all `n_results` regardless of how relevant they are? What are the tradeoffs of each approach?*

```
I will return all n_results for now instead of filtering by a hard threshold. This makes it easier to inspect retrieval behavior during testing. The tradeoff is that weak chunks may still be passed to generation, but since n_results is only 3, the context should stay small. If I see high-distance or unrelated chunks causing bad answers, I can add filtering later.
```

---

### Edge cases

*How does your implementation behave when: (a) the collection is empty, (b) the query matches no chunks well, (c) the query matches chunks from multiple games?*

```
If the collection is empty, retrieve() returns an empty list. If the query matches no chunks well, retrieve() still returns the closest available chunks, but their distance scores may be high and should be inspected during testing. If the query matches chunks from multiple games, retrieve() returns them ranked by distance because broad questions like "how do you win?" may reasonably apply to several games.
```

---

## Implementation Notes

*Fill this in after implementing, before moving to Milestone 3.*

**Test query and top result returned:**

```
Query: [your test query]
Top result game: [game name]
Distance score: [score]
Does it make sense? [yes / no / explain]
```

**One thing about the query results that surprised you:**

```
[your answer here]
```
