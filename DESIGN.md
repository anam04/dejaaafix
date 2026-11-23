# DESIGN

**Data model (SQLite).** `users`; `sessions` (user, pinned solution, escalated flag, summary, resolved);
`messages` (session, user, role, content, source); `solutions` (title, symptoms, environment, steps, did_not_work,
count, verified, session_id of origin); `solution_sessions` (every session that confirmed a solution);
`llm_log` (session, user, purpose, model, tokens in/out). Nothing is kept in memory, so a restart loses nothing.

**When the extractor runs, and why.** When a session ends (`end_session`). Confirmation comes after the fix, one chat
can hold several problems, and a single pass over the stored transcript is cheaper and more accurate than judging
every turn. It reads messages from the database. If the session was served from the hub, a tiny "was it confirmed?"
call is used instead of full extraction, which keeps hub sessions cheap.

**How a reworded problem finds its solution.** Each saved solution gets an embedding (Gemini) of its title,
symptoms and environment. A new problem is embedded too, and the 3 closest solutions by cosine similarity are
retrieved. The LLM then sees only those 3 short summaries and decides whether the problem is the *same fault*. I chose
embeddings because different wording shares few keywords, and the LLM check because embeddings alone cannot tell two
different faults in the same app apart (negative match). Only verified solutions are ever retrieved.

**How "confirmed" is detected.** The extractor must return a verbatim quote from a customer message sent after the
fix. Code checks the quote really exists in the stored messages. No valid quote, nothing is saved. Unconfirmed
chats are never stored. Personal data is removed by the prompt and again by a regex scrubber (emails, links, order
numbers, long numbers, the customer's name) before saving.

**Duplicates.** The extractor sees existing solutions and returns `duplicate_of`; a duplicate adds 1 to `count`
(once per session) instead of creating a copy.

**Assumptions.** Users are identified by an id passed in (no login). Rate limits: 20 messages/min and 5 escalations/hour
per user, raised as `RateLimited` (an API would return HTTP 429).

**With a week.** A real API (FastAPI) with 429 responses, a vector index for large hubs, a proper test suite,
tracking "this fix failed for me" to retire bad solutions, and auth.
