# Choir (Streamlit)

Streamlit web app that generates a “choir” from an uploaded vocal by sending preprocessing/inference jobs via AWS SQS and reading/writing audio assets in S3, then mixing results locally.

## What this project does

- **Input**: a vocal you upload in the UI.
- **Process**:
  - Uploads audio to **S3**
  - Sends **preprocessing** + **inference** requests via **SQS**
  - Waits for generated voices to appear in **S3**
  - Downloads generated stems and **mixes them locally** (pan/gain/time shift/chorus/tremolo + optional reverb)
- **Output**:
  - A mixed **choir WAV**
  - Optionally a **ZIP** with individual generated voice WAVs

## Run 

1. Create your environment file:

Create `.env` in the project root and provide at least `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, and `AWS_REGION`.

2. Edit `.env` and fill in at least:
```env
AWS_ACCESS_KEY_ID=...
AWS_SECRET_ACCESS_KEY=...
AWS_REGION=xx-xxxx-x
```

3. Create Streamlit login secrets (used by the UI):

Create `.streamlit/secrets.toml`:

```toml
username = "your_username"
password = "your_password"
```

4. Build + run:

```bash
docker compose up --build
```

5. Open the app:

- `http://localhost:8501`

### Stop / clean up

```bash
docker compose down
```

## How it works (high level)

- **Preprocessing step**: raw upload is written to S3, then the app sends a preprocessing message to SQS. The preprocessing service writes `_short.wav` and `_full.wav` back to S3.
- **Inference step**: for each selected voice/transposition, the app sends an inference message to SQS pointing at the (short/full) S3 input and an S3 output key to write to.
- **Polling**: the app polls S3 until all expected outputs exist (or timeouts).
- **Mixing**: generated voices are downloaded and mixed locally in the Streamlit container.

## Configuration

Defaults live in `src/config.py`:

- **AWS / storage**
  - `S3_BUCKET_NAME` (default: `voice-by-auribus-api`)
  - `S3_BASE_FOLDER` (default: `test_choir`)
- **Queues**
  - `SQS_QUEUE_PREPROCESSING`
  - `SQS_QUEUE_PAID_MAIN` (used when transposition is 0)
  - `SQS_QUEUE_PAID_ALT` (used when transposition is non-zero)
- **Timeouts / polling**
  - `PREPROCESSING_TIMEOUT` (default: `120`)
  - `POLL_TIMEOUT` (default: `600`)
  - `POLL_INTERVAL` (default: `2`)
- **UI / audio defaults**
  - `MIN_VOICES` (default: `2`), `MAX_VOICES` (default: `6`)

## Project structure

- `app.py`: Streamlit UI + orchestration (upload → preprocessing → inference → mixing)
- `src/aws.py`: S3/SQS I/O (upload/download, queue messages, local cache)
- `src/audio.py`: Mixing / audio rendering
- `src/tracks.py`: Track and voice management (voice configuration, incremental inference, DSP parameter sync)
- `src/config.py`: Defaults and env-var configuration
- `voice_previews/`: Local MP3 previews for available voice models

