# Movie Review Sentiment Analyzer

A sentiment analysis tool for movie reviews, built with Hugging Face Transformers and served through a Streamlit web app. The project includes a fine-tuned DistilBERT classifier, scripts for training/evaluating against a baseline model, and a small hand-labelled dataset of slang-heavy reviews for testing edge cases.

## Features

- **Streamlit web app** (`app.py`) with a simple UI for entering a review and getting a POSITIVE/NEGATIVE label with a confidence score.  
- **Fine-tuned model** (`fine_tuned_sentiment_model/`) — a DistilBERT model fine-tuned for binary sentiment classification (`NEGATIVE` / `POSITIVE`), based on the SST-2 task.  
- **Input validation & sanitization** — length checks, emoji-ratio checks, and basic filtering of URLs/script-like content before text reaches the model.  
- **Rate limiting** — caps each Streamlit session to a configurable number of requests per time window.  
- **Logging** — general application logs and a separate security event log (rate limiting, validation failures, suspicious input) are written per day.  
- **Baseline comparison script** (`sentiment_analyser.py`) — runs the public `distilbert-base-uncased-finetuned-sst-2-english` model against a sample of the IMDB test set and reports accuracy/classification metrics.  
- **Fine-tuned model evaluation script** (`sentiment_analyser_fine_tuned.py`) — loads the local fine-tuned model and runs the same kind of evaluation, plus an interactive CLI for testing custom text.  
- **Slang test set** (`slang_reviews.csv`) — 53 short, informal/slang-heavy movie reviews with sentiment labels, useful for stress-testing the model on casual language.

## Project structure

.

├── app.py                              \# Streamlit web app

├── functions.py                        \# Shared helpers: batch prediction & interactive CLI testing

├── sentiment\_analyser.py               \# Evaluates a public pretrained model on IMDB test data

├── sentiment\_analyser\_fine\_tuned.py    \# Loads and evaluates the local fine-tuned model

├── test.py                             \# Quick sanity check for environment variables

├── slang\_reviews.csv                   \# Small labelled dataset of slang/informal reviews

├── fine\_tuned\_sentiment\_model/         \# Fine-tuned DistilBERT model artifacts (not included — see Setup)

│   ├── config.json

│   ├── model.safetensors

│   ├── tokenizer.json

│   └── tokenizer\_config.json

├── requirements.txt                    \# Python dependencies

├── security\_YYYY-MM-DD.log             \# Daily security event log (generated at runtime)

└── streamlit\_app\_YYYY-MM-DD.log        \# Daily application log (generated at runtime)

## Setup

> **Note:** The fine-tuned model itself is **not** included in this repo/zip. You'll need to train (fine-tune) and export your own copy of the model before the app or evaluation scripts will run.  
> 

1. **Train and export the model.** Use the provided Google Colab notebook to fine-tune the model and export it in the correct format:  
     
   [Fine-tuning & export notebook →](https://colab.research.google.com/drive/1GWBY9BGcBjvH2AkPjVNm7638xBBfXUJD?usp=sharing)  
     
   Run through the notebook to train the model, then download the exported model folder (it will contain files like `config.json`, `model.safetensors`, `tokenizer.json`, and `tokenizer_config.json` — matching the `fine_tuned_sentiment_model/` structure shown below).  
     
2. **Create and activate a virtual environment** (a `venv/` folder is already present in this repo — reuse it or create a fresh one):  
     
   python \-m venv venv  
     
   source venv/bin/activate      \# Windows: venv\\Scripts\\activate  
     
3. **Install dependencies:**  
     
   pip install \-r requirements.txt  
     
4. **Move the exported model folder to a safe location** on your machine (it doesn't need to live inside this project folder, but it can).  
     
5. **Set the `MODEL_PATH` environment variable** to point at wherever you saved the exported model folder. The app and evaluation script both read this to load the model:  
     
   \# Windows (PowerShell)  
     
   $env:MODEL\_PATH \= "C:\\path\\to\\your\\fine\_tuned\_sentiment\_model"  
     
   \# macOS/Linux  
     
   export MODEL\_PATH="/path/to/your/fine\_tuned\_sentiment\_model"

## Usage

### Run the web app

streamlit run app.py

Then open the local URL Streamlit prints (typically `http://localhost:8501`) and enter a review to analyze.

### Evaluate the baseline pretrained model

python sentiment\_analyser.py

Downloads a sample of the IMDB test set, runs it through `distilbert-base-uncased-finetuned-sst-2-english`, and prints accuracy and a classification report. Ends with an interactive prompt for testing your own text.

### Evaluate the fine-tuned model

python sentiment\_analyser\_fine\_tuned.py

Loads the model from `MODEL_PATH`, runs the same IMDB-based evaluation, and drops into an interactive prompt where you can type reviews and see predictions live (type `quit` to exit).

### Quick environment check

python test.py

Prints the current `MODEL_PATH` and `home` environment variables — useful for confirming your environment is set up correctly before running the other scripts.

## Model details

- **Base architecture:** DistilBERT (`distilbert-base-uncased`)  
- **Task:** Binary sentiment classification (`sst-2`\-style)  
- **Labels:** `0` \= NEGATIVE, `1` \= POSITIVE  
- **Max sequence length:** 512 tokens  
- **Training:** Fine-tuned via the [Colab notebook linked above](https://colab.research.google.com/drive/1GWBY9BGcBjvH2AkPjVNm7638xBBfXUJD?usp=sharing) — run it to reproduce the model or fine-tune your own version.

## Logging & security

The Streamlit app writes two categories of logs to the project root, named by date:

- `streamlit_app_YYYY-MM-DD.log` — general application activity (model loading, request counts, errors).  
- `security_YYYY-MM-DD.log` — security-relevant events such as rate limit breaches, oversized input, and suspicious content (URLs, script tags) in submitted reviews.

Input to the app is validated (length, emoji ratio, URL/script detection) and sanitized (HTML tag stripping, repeated-character collapsing) before being passed to the model. Each session is limited to 10 requests per 60-second window by default; these values can be adjusted via the `RATE_LIMIT_MAX` and `RATE_LIMIT_WINDOW` constants in `app.py`.

## Notes

- `requirements.txt` lists `logging`, which is part of the Python standard library and doesn't need to be installed separately — it can be safely removed from that file if you want to trim it down.  
- Log files (`*.log`) are generated at runtime and will accumulate in the project directory; consider adding them to `.gitignore` if this repo is put under version control.

