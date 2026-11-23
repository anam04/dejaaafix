

One file (`dejafix.py`), SQLite, runs in Google Colab.

## Run in Colab:
1. Upload `dejafix.py` to Colab (folder icon on the left).
2. Cell 1: `!pip -q install google-genai`
3. Cell 2: `%run dejafix.py` - it asks you to paste your Gemini API key (hidden box, never saved in the file).
   It then runs the 3-session demo on an empty database and prints the token log.
4. Restart test: restart the runtime, run `import dejafix; print(len(dejafix.verified_solutions()))`.
   The saved solution is still there (file `dejafix.db`).

Get a key at aistudio.google.com. Never paste it into the code or push it to GitHub.
Models: `gemini-3.5-flash-lite` (chat) 
## Using it yourself
```python
import dejafix as d
d.init_db(); d.add_user("u1", "Your Name")
sid, hello = d.start_session("u1")
print(d.chat("u1", sid, "my problem..."))
print(d.end_session("u1", sid))      # runs the extractor, saves only confirmed fixes
d.read_session(sid); d.list_sessions("u1"); d.token_report()
```
