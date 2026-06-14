# Spec: `generate_response()`

**File:** `generator.py`
**Status:** Spec incomplete — fill in all blank fields before implementing

---

## Purpose

Given a user query and a list of retrieved rule chunks, generate a response that directly answers the question using only the retrieved text as context. The response must be grounded — it should not draw on the model's general knowledge of board games, only on what was retrieved.

---

## Input / Output Contract

**Inputs:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `query` | `str` | The user's original question |
| `retrieved_chunks` | `list[dict]` | Ranked list of chunks from `retrieve()`, each with `"text"`, `"game"`, and `"distance"` |

**Output:** `str`

A plain string containing the response to show the user. The response should:
- Answer the question using only the retrieved rule text
- Identify which game the answer comes from
- Acknowledge clearly when the answer is not found in the loaded rules

Returns a fallback string (not an error) when `retrieved_chunks` is empty.

---

## Design Decisions

*Complete the fields below before writing any code. Use your AI tool in Plan or Ask mode to help you reason through what belongs here — but the decisions are yours.*

---

### Context formatting

*How will you format the retrieved chunks before passing them to the LLM? Describe the structure — not the code. Consider: will you label chunks by game? Include distance scores? Separate chunks with delimiters?*

```
I will format each retrieved chunk with a clear chunk number, game label, distance score, and rule text. Each chunk will be separated with delimiters so the model can tell where one source ends and the next begins. Including the game name helps the model cite which game the answer comes from.
```

---

### System prompt — grounding instruction

*Write the exact system prompt instruction you will use to prevent the model from answering beyond the retrieved text. This is the most important design decision in this function.*

```
You are RulesBot, a board game rules assistant. Answer using only the retrieved rule text provided by the user. Do not use outside knowledge, prior knowledge, assumptions, or guesses. If the retrieved rule text does not contain the answer, say that the loaded rule books do not contain enough information to answer.
```

---

### System prompt — citation instruction

*Write the exact instruction you will use to tell the model to identify which game its answer comes from.*

```
Every answer must identify the game or games the answer came from. Use wording like "According to the Catan rules..." when the source game is clear.
```

---

### Fallback behavior

*What should the response say when the answer isn't found in the loaded rule books? Write the exact fallback message.*

```
I couldn't find enough information in the loaded rule books to answer that.
```

---

### Handling low-relevance chunks

*`retrieved_chunks` may include chunks with high distance scores (weak relevance). Will you filter these out before building context, pass them all in, or handle them another way? What are the tradeoffs?*

```
I will pass all retrieved chunks into the prompt for now because retrieve() only returns the top 3 results. The benefit is that the model gets all available context. The risk is that a weakly related chunk could distract the model. If testing shows that high-distance chunks are causing wrong answers, I would filter out chunks above a distance threshold before generation.
```

---

### Message structure

*Describe how you will structure the messages list for the API call — what goes in the system message vs. the user message?*

```
The system message will contain the grounding and citation instructions. The user message will contain the formatted retrieved chunks followed by the user's original question. This separates behavior instructions from the actual task content.
```

---

## Implementation Notes

*Fill this in after implementing and testing.*

**Test query and response:**

```
Query: How do you set up the board in Catan?
Response: According to the Catan rules, to set up the board, you arrange the terrain hexes randomly...
Correctly grounded? yes
Cited the right game? yes
```

**One thing you changed from your original spec after seeing the actual output:**

```
I did not change the original spec after testing. The grounding instruction and chunk formatting worked as expected, so I kept the implementation aligned with the original plan.
```
