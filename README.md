[README_FinBERT.md](https://github.com/user-attachments/files/27426621/README_FinBERT.md)
# 🧠 Financial Sentiment Analysis with FinBERT

A multi-source financial sentiment pipeline that scores stock tickers using **FinBERT** — a BERT model fine-tuned on financial text — by pulling and analyzing headlines and filings from Finviz, Google News RSS, and the **SEC EDGAR API**. Outputs a per-ticker sentiment label and score exported to CSV.

---

## Overview

Generic sentiment tools like VADER are trained on general language and miss the nuance of financial text — a phrase like "missed estimates" is neutral in everyday speech but strongly negative in finance. This project uses **FinBERT** (`yiyanghkust/finbert-tone`), a domain-specific model trained on financial communications, to classify text as positive, negative, or neutral with higher accuracy than rule-based alternatives.

Given a list of tickers, the pipeline:
1. Scrapes recent headlines from **Finviz** and **Google News RSS**
2. Fetches and parses the two most recent **SEC 8-K filings** from EDGAR
3. Scores every text snippet through FinBERT
4. Aggregates scores into a single sentiment label per ticker
5. Exports results to a CSV file

---

## Data Sources

| Source | What It Provides | Method |
|---|---|---|
| **Finviz** | Latest news headlines from the stock's quote page | BeautifulSoup HTML scrape |
| **Google News RSS** | Recent news articles matching `[TICKER] stock` | `feedparser` RSS parse |
| **SEC EDGAR (8-K)** | Material event filings (earnings, acquisitions, guidance) | SEC EDGAR REST API + BeautifulSoup |

### Why 8-K Filings?
8-K filings are material event disclosures — earnings releases, M&A announcements, executive changes, guidance updates. They represent the most market-moving corporate language available in structured, publicly accessible form. Including them grounds the sentiment score in official company communications, not just media interpretation.

---

## Sentiment Scoring

Each text snippet is classified by FinBERT into one of three labels and converted to a numeric score:

| FinBERT Label | Numeric Score |
|---|---|
| `positive` | `+confidence` (e.g., +0.94) |
| `negative` | `−confidence` (e.g., −0.87) |
| `neutral` | `+0.01` |

Scores are averaged across all sources for a final composite score per ticker.

---

## Signal Labels

| Average Score | Label |
|---|---|
| > 0.10 | **Buy** |
| −0.10 to 0.10 | **Hold** |
| −0.40 to −0.10 | **Sell** |
| < −0.40 | **Strong Sell** |

---

## Sample Output

```
Ticker    Average Score    Sentiment Label
NVDA         0.3241            Buy
AAPL         0.0812            Hold
MSFT         0.1947            Buy
```

Saved to `sentiment_regression_data_2025.csv`.

---

## Getting Started

**1. Clone the repo**
```bash
git clone https://github.com/your-username/finbert-sentiment.git
cd finbert-sentiment
```

**2. Install dependencies**
```bash
pip install yfinance transformers feedparser beautifulsoup4 pandas requests
```

**3. Run the notebook**
```bash
jupyter notebook Sentimental_Analysis_Finbert.ipynb
```

When prompted:
```
Enter comma-separated stock tickers (e.g., NVDA, AAPL, MSFT):
```

The pipeline will run automatically and print per-headline scores before saving the summary CSV.

---

## SEC EDGAR Compliance

The notebook uses a compliant `User-Agent` header as required by the SEC for EDGAR API access:

```python
HEADERS = {
    "User-Agent": "Name Institution email@domain.edu",
    "Accept-Encoding": "gzip, deflate"
}
```

Update this header with your own name and email before running to remain compliant with SEC data access policies.

---

## FinBERT Model

| Detail | Value |
|---|---|
| Model | `yiyanghkust/finbert-tone` |
| Base | BERT (bert-base-uncased) |
| Fine-tuned on | Financial news, earnings call transcripts, analyst reports |
| Labels | `positive`, `negative`, `neutral` |
| Token limit | 512 tokens (headlines exceeding this are truncated before scoring) |

FinBERT significantly outperforms VADER on financial text because it was trained specifically on the language patterns used in earnings calls, SEC filings, and financial journalism — domains where word choice carries precise implications that general sentiment models miss.

---

## Tech Stack

| Library | Purpose |
|---|---|
| `transformers` | FinBERT model loading and inference pipeline |
| `requests` | Finviz and SEC EDGAR HTTP requests |
| `beautifulsoup4` | HTML parsing for Finviz and SEC filing text |
| `feedparser` | Google News RSS feed parsing |
| `pandas` | Results aggregation and CSV export |
| `yfinance` | Available for price data enrichment |

---

## Project Structure

```
finbert-sentiment/
├── Sentimental_Analysis_Finbert.ipynb   # Full pipeline
├── sentiment_regression_data_2025.csv  # Auto-generated output
└── README.md
```

---

## Potential Extensions

- **10-K filings** — add annual report parsing alongside 8-Ks for a longer-horizon sentiment view covering risk factors and MD&A sections
- **Earnings call transcripts** — scrape or source transcripts from Seeking Alpha or Motley Fool for high-signal executive language
- **Time-series sentiment** — store daily scores over time and correlate sentiment trends with price movements
- **Score-to-return regression** — use the exported CSV as features in a regression model to quantify how much predictive power sentiment scores carry relative to price returns
- **Streamlit dashboard** — wrap the pipeline in a live UI where users enter tickers and get real-time sentiment cards

---

## Disclaimer

This project is for educational and research purposes only. Sentiment scores are not financial advice. Always conduct independent research before making investment decisions.

---

## License

MIT License. See `LICENSE` for details.
