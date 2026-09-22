# MP1 Prompt Lab — Running & Interpreting Results

## Prerequisites

- Python 3.10+
- An OpenAI API key with access to `gpt-4o-mini` and `gpt-4o`
- Jupyter (Lab or classic Notebook) installed

---

## Setup

### 1. Install dependencies

From the repo root:

```bash
pip install -r requirements.txt
```

Or, if working inside `mp1/` directly:

```bash
pip install openai pandas jupyter
```

### 2. Set your API key

The notebook asserts the key is present before making any calls.

**Mac/Linux:**
```bash
export OPENAI_API_KEY=sk-...
```

**Windows (PowerShell):**
```powershell
$env:OPENAI_API_KEY = "sk-..."
```

Or add it to a `.env` file and load it with `python-dotenv` before launching Jupyter.

### 3. Verify data files exist

The notebook expects these two files relative to `mp1/`:

```
mp1/
  data/
    job_snippets.jsonl   ← 10 job posting snippets
    golden_set.jsonl     ← ground-truth extractions
  mp1_prompt_lab.ipynb
```

If either file is missing the data-loading cell will error immediately.

---

## Running the Notebook

### Launch Jupyter

```bash
cd mp1
jupyter lab mp1_prompt_lab.ipynb
```

### Run all cells in order

Use **Run → Run All Cells** (or `Shift+Enter` cell by cell). The cells must run in order — later
cells depend on variables set by earlier ones.

| Cell | What it does |
|------|-------------|
| **Setup** | Imports, sets model names, cost rates |
| **Step 1** | Loads snippets and golden set into memory |
| **Step 2** | Defines the four prompt functions |
| **Step 3** | Fires all 40 API calls (`await run_all()`) |
| **Step 4** | Scores each result (accuracy, parse rate, LLM judge) |
| **Step 5** | Builds and prints the summary comparison table |
| **Step 6** | Reflection prompt (no code — write your writeup here) |

> **Note:** Steps 3 and 4 make real API calls. Step 3 runs 40 calls in parallel; Step 4 runs
> another 40 judge calls via `gpt-4o`. Expect ~5–10 seconds total and a combined cost of roughly
> **$0.01–0.02** per full run.

---


## Results

- `mp1_comparison.md` contains the experiment results and strategy comparison.
- `mp1_writeup.md` contains the reflection answers.