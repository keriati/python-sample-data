# Coding Exercise — Source-Aware QA Agent

Welcome! In this exercise, you’ll build a small, source-aware question-answering service. The tasks build on each other step by step. Please aim for clear, modular code, with type hints, docstrings, and tests.

We recommend Python 3.11+, and the following libraries:

* [LangChain](https://python.langchain.com) (for runnables, tools, and agents)
* [FastAPI](https://fastapi.tiangolo.com/) and Uvicorn (for the API + SSE streaming)
* [Pydantic](https://docs.pydantic.dev/) v2 (for models/validation)
* [SQLAlchemy](https://www.sqlalchemy.org/) (for state persistence)
* A vector store (FAISS or Chroma)
* `pytest` (for testing)

You may use any LLM you can access (OpenAI, LiteLLM, or a small local model). Please abstract the LLM so we can swap providers easily.

We’ll provide a small set of text passages in `./data/` to work with. Each passage ends with a **source line** like:

```
Some short passage text goes here...

Source: Title | Author | 2022-04-01 | https://example.com
```

---

## Task 1 — Source extractor runnable

* Implement a LangChain **Runnable** that extracts structured metadata from a passage.
* Expected output model:

  ```python
  class SourceInfo(BaseModel):
      title: str
      author: str
      date: date
      url: Optional[HttpUrl] = None
  ```
* Handle missing fields gracefully.
* Add simple unit tests (well-formed input, missing URL, malformed date, no “Source:” line).

---

## Task 2 — Tool + ReAct agent

* Wrap your runnable as a LangChain **Tool** named `"extract_source"`.
* Build a minimal **ReAct agent** that can:

    * Answer questions about a passage.
    * Call the `extract_source` tool when it needs source details.
* The agent’s final answer should include a **Sources** section listing title, author, date.

---

## Task 3 — Retrieval tool + multi-tool agent

* Write an ingestion script to:

    * Read the files in `./data/`.
    * Chunk them (500–1000 tokens).
    * Store embeddings in a vector store.
* Implement a `"retrieve_passages"` tool:

    * Args: `{ "query": str, "k": int=4 }`
    * Returns: list of `{ "passage": str, "doc_id": str }`
* Extend the agent so it can:

    * Call `retrieve_passages` to get relevant passages.
    * Optionally call `extract_source` on them.
    * Answer the user question and **cite the correct source**.

---

## Task 4 — FastAPI service with SSE

* Build a FastAPI app with one endpoint:

    * `POST /chat` → **SSE response** (`text/event-stream`)
* Request:

  ```json
  { "session_id": "string", "message": "user question" }
  ```
* Response:

    * Stream tokens as `event: token`.
    * Finish with `event: done` containing usage info and the list of cited sources.
* Provide a minimal static HTML page (or curl instructions) to demo streaming.

---

## Task 5 — Persistent state

* Add a SQLite DB (or similar) to track:

    * Sessions
    * Messages
    * Runs
    * Tool calls
* On each `/chat` call:

    * Save the run, user message, assistant tokens, and tool calls.
* Add an endpoint `GET /sessions/{session_id}` to return a transcript with tool call summaries.

---

## Deliverables

* Working repo with the code, `README.md` (setup & run instructions), and your `TASKS.md`.
* `pytest -q` runs successfully without network access (except optional LLM calls if mocked).
* Code that is clean, typed, and well-structured.

---

👉 You don’t need to make everything production-ready. Focus on correctness, clarity, and showing how you structure and test code.

Good luck, and have fun! 🚀
