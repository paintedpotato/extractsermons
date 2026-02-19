# YouTube Channel Transcript Extractor & Keyword Ranker

A Python pipeline for extracting **auto-generated captions** from an entire YouTube channel, converting them into a structured dataset, and generating keyword-based analytics (example: ranking all mentions of “Mary” with timestamps and video URLs).

This project is useful for:
- Research (theology, lectures, sermons, education)
- NLP dataset preparation
- Content mining from large video archives
- LLM training workflows

The workflow runs locally or in :contentReference[oaicite:0]{index=0}.

---

## Features

- Automatically extract **all videos** from a channel  
- Download **auto captions** (same captions used by :contentReference[oaicite:1]{index=1} player)  
- Convert `.vtt` subtitle files into structured rows  
- Combine all transcripts into a single CSV  
- Keyword detection with:
  - timestamps
  - video URLs
  - frequency ranking per video
- Progress bars for:
  - video processing
  - caption parsing
  - keyword scanning

---

## Pipeline Overview

Channel URL→Fetch video IDs→Download auto captions→Parse VTT subtitle files→Combine transcripts into dataset→Keyword ranking (example: "Mary")

````

---

## Requirements

Install dependencies:

```bash
pip install yt-dlp pandas tqdm
````

This project uses:

* yt-dlp for caption extraction
* pandas for dataset processing
* tqdm for progress tracking

---

## Quick Start (Google Colab Recommended)

### 1. Install dependencies

```python
!pip install yt-dlp pandas tqdm
```

---

### 2. Set channel URL

```python
CHANNEL_URL = "YOUR_CHANNEL_URL"
```

Run the extraction pipeline to generate:

```
all_transcripts.csv
```

---

### 3. Run keyword extraction

The keyword script scans transcripts and produces:

```
mary_mentions_ranked.csv
```

Output columns:

| video_id | video_url | mary_mentions | timestamp | text |
| -------- | --------- | ------------- | --------- | ---- |

---

## Output Files

### all_transcripts.csv

Combined captions from all videos.

```
video_id,timestamp,text
```

---

### mary_mentions_ranked.csv

All keyword occurrences ranked by frequency per video.

```
video_id,video_url,mary_mentions,timestamp,text
```

---

## Customization

### Change keyword

Replace:

```python
"mary"
```

with any keyword:

```python
"moses"
"faith"
"grace"
```

Regex patterns are supported.

---

### Change subtitle language

```python
LANG = "en"
```

Examples:

```
es  Spanish
fr  French
sw  Swahili
```

---

## Project Structure

```
project/
│
├─ transcripts/
│   ├─ VIDEOID.en.vtt
│
├─ all_transcripts.csv
├─ mary_mentions_ranked.csv
└─ README.md
```

---

## Limitations

* Only videos with available auto captions can be processed.
* Auto captions may contain transcription errors.
* Large channels may take longer to process.

---

## Recommended Extensions

* Multi-keyword ranking
* Context window extraction (±2 lines around keywords)
* Video metadata enrichment (title, publish date)
* Semantic search indexing
* LLM dataset generation

---

## License

MIT License — free to use and modify.

---

## Summary

This project creates a fast pipeline to:

1. Extract captions from a full YouTube channel
2. Convert them into a structured dataset
3. Rank keyword mentions with timestamps and video links

Ideal for large-scale transcript mining and research workflows.

```
```
