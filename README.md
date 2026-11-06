# YouTube Channel Transcript Extractor & Keyword Ranker

A Python pipeline that pulls the **auto-generated captions** from a YouTube channel's videos and livestreams from the **past 10 years**, turns them into a structured dataset, and ranks every video by how often a phrase is said. The current search is for **"I am"**, with timestamps, surrounding text and video links.

This project is useful for:
- Research (theology, lectures, sermons, education)
- NLP dataset preparation
- Content mining from large video archives
- LLM training workflows

It is built for **Google Colab**. It also runs in Jupyter if you turn off the Colab-only parts (Google Drive, GitHub push, downloads).

---

## Features

- Lists every video on the channel's **Videos** and **Live** (past livestreams) tabs, plus **Shorts** if you add that tab
- Keeps only videos from the **last 10 years**, using each video's exact upload date
- Downloads **auto captions** (the same captions shown in the YouTube player)
- Removes the repeated lines in YouTube's auto-captions, so mentions aren't counted twice
- Combines all transcripts into one CSV with year, upload date and title
- Phrase search that:
  - ignores capitalisation ("I am", "I AM")
  - still finds the phrase when it is split across two caption lines
  - records the timestamp, surrounding text and video URL
  - ranks videos by how many times the phrase appears
- **One progress bar per year** for downloading captions, parsing them and searching them
- **Picks up where it left off** after a crash: progress is saved to Google Drive every 10 videos
- Optional cells to **push the results to GitHub** and **download everything as a zip**

---

## Pipeline Overview

Channel URL → List videos and livestreams with dates → Keep the last 10 years → Download auto captions (saved as it goes) → Parse VTT files and remove repeated lines → Combined transcript dataset → Search for "I am" and rank videos

---

## Requirements

```bash
pip install yt-dlp pandas tqdm
```

- **yt-dlp:** lists the videos and downloads the captions
- **pandas:** builds and processes the dataset
- **tqdm:** shows the progress bars

---

## Quick Start (Google Colab)

Paste each `##NEW SECTION` of the notebook into its own cell and run them in order.

| Section | What it does |
| --- | --- |
| 0 | Installs the packages and mounts Google Drive. All files are saved in `MyDrive/tmi_i_am`. |
| 1 | Lists the channel's videos and livestreams and keeps the last 10 years. The list is saved and reused when you resume. |
| 2 | Downloads the captions, with one progress bar per year and a save every 10 videos. Also fills in each video's exact upload date. |
| 3 | Parses the `.vtt` files into `all_transcripts.csv`, with one progress bar per year. |
| 4 | Searches for "I am" and writes the ranked CSVs, with one progress bar per year. |
| 5 | *(Optional)* Pushes the result CSVs to a GitHub repository. |
| 6 | *(Optional)* Downloads everything, including the `transcripts` folder, as one zip. |

Set the channel in section 1:

```python
CHANNEL_URL = "https://www.youtube.com/channel/YOUR_CHANNEL_ID"
```

---

## Resuming After a Crash

All progress is kept in your Google Drive folder (`DRIVE_DIR` in section 0). If Colab disconnects or the runtime stops:

1. Open the notebook again and run all cells from the top.
2. Section 1 loads the saved video list instead of fetching it again.
3. Section 2 skips videos that are already done and starts each year's progress bar from where it stopped. Videos that failed are tried again.

Each video's status is saved in `videos.csv`:

| Status | Meaning |
| --- | --- |
| `ok` | Captions downloaded |
| `no_captions` | The video has no English captions |
| `error` | The download failed; it is retried on the next run |

To check the channel for new uploads without losing progress, set `REFRESH_LIST = True` in section 1.

If you can't use Google Drive, set `USE_DRIVE = False`. Then use the optional backup cell to download a progress zip, and its restore lines to upload it into a new session.

---

## Output Files

### i_am_by_video.csv

One row per video, ranked by the number of mentions. This is the main ranking.

```
rank,year,upload_date,video_id,title,video_url,i_am_mentions
```

### i_am_mentions_ranked.csv

Every mention, in the same video order as the ranking, with where it happens and what was said around it.

```
rank,year,upload_date,video_id,title,video_url,i_am_mentions,timestamp,context
```

Videos with the same number of mentions share a rank. The notebook also prints the number of mentions and videos for each year.

### all_transcripts.csv

All captions combined, with repeated lines removed.

```
year,upload_date,video_id,title,timestamp,text
```

### videos.csv

The video list with upload dates, the tab each video came from, and its download status. This is the file that makes resuming possible.

---

## Customization

### Include "I'm"

In section 4:

```python
INCLUDE_IM = True   # also counts "I'm" / "I’m"
```

### Change the phrase

Edit `PHRASE` in section 4. It is a regular expression that ignores capitalisation:

```python
PHRASE = r"\bmary\b"
PHRASE = r"\bholy\s+spirit\b"
```

You may also want to rename the output files and the `i_am_mentions` column.

### Change the time range

```python
YEARS_BACK = 10
```

### Choose which tabs to scan

```python
TABS = ["videos", "streams", "shorts"]
```

### Change the subtitle language

```python
LANG = "en"   # es Spanish, fr French, sw Swahili
```

### Context length

`CONTEXT_CHARS = 60` sets how much text is kept on each side of a mention.

---

## Push Results to GitHub (section 5)

1. Create a GitHub personal access token with **Contents: write** access to your repository.
2. In Colab, open 🔑 **Secrets**, add the token as `GITHUB_TOKEN`, and turn on notebook access.
3. Set `GITHUB_REPO = "your-username/your-repo"` (the repository must already exist).

The cell pushes `i_am_mentions_ranked.csv`, `i_am_by_video.csv` and `videos.csv`. `all_transcripts.csv` is left out because it can go over GitHub's 100 MB file limit. To save the notebook itself, use **File → Save a copy in GitHub**.

---

## Download Everything (section 6)

```python
import shutil; from google.colab import files
files.download(shutil.make_archive("/content/i_am_outputs", "zip", "."))
```

This zips the whole working folder, including all CSVs and the `transcripts` folder of `.vtt` files.

---

## Project Structure

```
tmi_i_am/                       (in Google Drive)
│
├─ transcripts/
│   └─ VIDEOID.en.vtt
│
├─ videos.csv                   video list + progress
├─ all_transcripts.csv
├─ i_am_mentions_ranked.csv
├─ i_am_by_video.csv
└─ README.md
```

---

## Limitations

- Only videos with captions can be searched. Videos without them are marked `no_captions`.
- Auto captions contain transcription errors, so some mentions may be missed or wrong.
- The first video listing uses approximate dates. Exact dates are filled in during the caption download, so the yearly counts in section 1 are only estimates until section 2 finishes.
- Large channels take a while because each video needs its own request. Resuming means you never lose finished work.
- YouTube may slow down or block requests from Colab during very long runs. If that happens, wait and re-run section 2.

---

## Recommended Extensions

- Search for several keywords at once
- Semantic search indexing
- LLM dataset generation
- Charts of mentions per year

---

## License

MIT License — free to use and modify.

---

## Summary

This project:

1. Extracts captions from a YouTube channel's videos and livestreams from the last 10 years
2. Turns them into a structured, repeat-free dataset grouped by year
3. Ranks videos by how often "I am" is said, with timestamps, surrounding text and links
4. Picks up where it left off after a crash, and can push to GitHub or download everything as a zip
