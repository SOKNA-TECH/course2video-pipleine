# Video-to-Course Pipeline

Turn a folder of long training/onboarding videos into a structured, web-ready microlearning course — complete with transcripts, topic segmentation, and optimized video clips.

Everything runs on **Google Colab** (no local GPU or heavy downloads needed). Just plug in your API keys and Google Drive folder links.

> **How it works:** The notebooks are lightweight orchestrators that manage the workflow and data flow. Heavy processing (ASR transcription, LLM inference) happens on the API providers' servers — not on Colab's runtime. Colab just coordinates the calls and handles file I/O with Google Drive.

## Pipeline

```
┌─────────────┐   ┌──────────────┐   ┌───────────────┐   ┌───────────────┐   ┌──────────────┐
│ 01 Extract   │   │ 02 Transcribe│   │ 03 Semantic   │   │ 04 Video      │   │ 05 Articles  │
│ Audio        │──>│ with ASR     │──>│ Chunking      │──>│ Segmentation  │   │ & Quizzes    │
│              │   │              │   │               │   │               │   │              │
│ Videos → WAV │   │ GCS + Qwen3  │   │ LLM → topics  │   │ FFmpeg cut    │   │ LLM → content│
└─────────────┘   └──────┬───────┘   └───────────────┘   └───────────────┘   └──────────────┘
                         │                                                          ▲
                         └──────────────────────────────────────────────────────────┘
                              Step 5 reads directly from Step 2 transcripts
```

**Data flow:**

```
Google Drive Videos (.mp4)
    │
    ▼
WAV Audio (16kHz mono) ─── via GCS public URL ──→ Qwen3-ASR
    │                                                  │
    │                                                  ▼
    │                                          Transcript JSONs
    │                                          (full text + sentence timestamps)
    │                                            │              │
    │                                            ▼              ▼
    │                                    LLM Chunking     LLM Article +
    │                                    Plans            Quiz Generation
    │                                    (Step 3)         (Step 5)
    │                                            │              │
    ▼────────────────────────────────────────────▼              ▼
                        FFmpeg                          *_article.json
                          │                             *_quiz.json
                          ▼
                   Final Output:
                   ├── Day_1/
                   │   ├── Video_Title/
                   │   │   ├── Video_Title.json        (full transcript)
                   │   │   ├── Video_Title_article.json (article summary)
                   │   │   ├── Video_Title_quiz.json   (quiz questions)
                   │   │   ├── 01_introduction/
                   │   │   │   ├── video.mp4           (720p web clip)
                   │   │   │   └── data.json           (title, summary, sentences)
                   │   │   ├── 02_core_concepts/
                   │   │   │   ├── video.mp4
                   │   │   │   └── data.json
                   │   │   └── ...
```

## Quick Start

1. **Clone the repo**
   ```bash
   git clone https://github.com/YOUR_USERNAME/video-to-course-pipeline.git
   ```

2. **Open any notebook in Google Colab**

   Click the "Open in Colab" badges below, or upload the notebooks manually.

3. **Configure your keys**

   Each notebook has a Configuration cell at the top. Fill in your API keys and Google Drive paths. See `config/.env.example` for a full reference of all required values.

4. **Run the notebooks in order**

   `01` → `02` → `03` → `04` → `05`. Steps 3-4 (chunking + video) and Step 5 (articles + quizzes) both read from Step 2's transcript output, so they can run in parallel or in any order after Step 2.

## Notebooks

| # | Notebook | What it does | Services used |
|---|----------|-------------|---------------|
| 01 | `01_extract_audio.ipynb` | Extracts 16kHz mono WAV from video files | FFmpeg (pre-installed in Colab) |
| 02 | `02_transcribe_asr.ipynb` | Uploads audio to GCS, transcribes via Qwen3-ASR | Google Cloud Storage, Alibaba DashScope |
| 03 | `03_semantic_chunking.ipynb` | LLM identifies topic boundaries and creates a learning plan | Any OpenAI-compatible API (default: Kimi K2) |
| 04 | `04_video_segmentation.ipynb` | Cuts videos into chunks, builds structured output | FFmpeg, (optional) S3-compatible upload |
| 05 | `05_articles_and_quizzes.ipynb` | Generates article summaries and multiple-choice quizzes | Any OpenAI-compatible API (default: Kimi K2) |

## Platform/library choices

Our current choices are the following for convenience and preference but also, you can adapt to your own preferences easily.

- **Google Cloud Platform** — a GCP project ID with Cloud Storage API enabled. No API key needed — Colab's built-in `auth.authenticate_user()` handles authentication via an OAuth popup. The project ID is only used to create the temporary staging bucket.
- **Alibaba Cloud / DashScope** — API key for Qwen3-ASR transcription ([sign up](https://www.alibabacloud.com/en/solutions/generative-ai/dashscope))
- **LLM API** — any OpenAI-compatible provider. Default is Kimi K2 via [Moonshot](https://platform.moonshot.cn/), but GPT-4, Claude, Gemini, etc. all work by changing `BASE_URL` and `MODEL_NAME`
- **(Optional) S3-compatible storage** — for uploading final output (AWS S3, Railway, Cloudflare R2, MinIO, etc.)

> **Privacy Note:** Your data is sent to API providers (Alibaba DashScope for ASR, LLM providers like Kimi) as paid API calls. Providers have terms of service protecting your data and do not train on your content without consent. The notebooks only orchestrate the workflow — heavy processing happens on the API providers' servers.

## Data Schemas

See the `schemas/` folder for sample JSON files showing the exact structure produced at each stage:

- `transcript_output.json` — what Qwen3-ASR returns (Step 2 output)
- `chunking_plan.json` — the LLM's segmentation plan (semantic chunking) (Step 3 output)
- `article_output.json` — the LLM-generated article summary (Step 5 output)
- `quiz_output.json` — the LLM-generated quiz with per-option feedback (Step 5 output)

## Tips and Lessons Learned

**Why extract audio instead of sending video directly?**
Qwen3-ASR can process video files directly, but our videos were around 2 GB each (40 GB in total). Extracting audio-only WAV files drastically reduces the upload size and processing cost. The entire pipeline (5 days of training content) cost less than $2 in ASR fees using audio-only files and batch processing.

**Why Qwen3-ASR instead of Whisper?**
For Arabic and mixed-language content, Qwen3-ASR produced significantly better results. It also provides sentence-level timestamps out of the box, which Whisper's default mode doesn't. The file transcription API handles long recordings without chunking on your side. We use the `qwen3-asr-flash-filetrans` model which is optimized for long/batch file transcription, but DashScope offers other ASR models if you need different speed/quality tradeoffs.

**Why do we need a public bucket? Why not just use Google Drive links?**
Alibaba Cloud's ASR API requires a publicly accessible URL with proper HTTP headers. Google Drive download links don't work, they redirect through consent pages and serve HTML instead of the actual file. The solution is to upload to any public bucket (GCS, S3, R2, etc.), get a direct URL, and delete the bucket when done. We use GCS because Colab has native GCP authentication built in, but any S3-compatible bucket works. The GCP project ID is only used to create this temporary bucket. No API key is needed, just the OAuth popup.

**Why Kimi K2 Thinking for Semantic Chuncking?**
It handles large JSON payloads well and follows the structured output format reliably at low temperature. But any strong model works — the system prompt does the heavy lifting. Just swap `BASE_URL`, `API_KEY`, and `MODEL_NAME`.

**Why upload final videos to S3 instead of serving from Google Drive?**
Google Drive doesn't support proper video streaming, it either shows a limited iframe player or forces a download. For a web-based course player, you need direct video URLs with proper streaming headers. The scripts handles generating the optimised videos. Uploading the optimized clips to an S3-compatible bucket (AWS S3, Railway, Cloudflare R2, etc.) gives you direct URLs with `+faststart` metadata that work with any HTML5 video player.

**FFmpeg flags that matter:**
- `-crf 28` — good compression vs. quality tradeoff for training content
- `-movflags +faststart` — critical for web streaming (moves metadata to the start of the file)
- `-vf scale=-1:720` — 720p is plenty for talking-head training videos
- `-ac 1 -ar 16000` — mono 16kHz is the native format for most ASR models


## Project Structure

```
video-to-course-pipeline/
├── README.md
├── .gitignore
├── LICENSE
├── config/
│   └── .env.example          # All configurable values in one place
├── notebooks/
│   ├── 01_extract_audio.ipynb
│   ├── 02_transcribe_asr.ipynb
│   ├── 03_semantic_chunking.ipynb
│   ├── 04_video_segmentation.ipynb
│   └── 05_articles_and_quizzes.ipynb
└── schemas/
    ├── transcript_output.json  # Sample Step 2 output
    ├── chunking_plan.json      # Sample Step 3 output
    ├── article_output.json     # Sample Step 5 output (article)
    └── quiz_output.json        # Sample Step 5 output (quiz)
```

## License

MIT
