# Choir (Streamlit)

Streamlit web app that generates a “choir” from an uploaded vocal by sending preprocessing/inference jobs via AWS SQS and reading/writing audio assets in S3, then mixing results locally.

## Run with Docker + `.env` (recommended)

1. Create your environment file:
Create .env file and provide AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY and AWS_REGION.

2. Edit `.env` and fill in at least:
```env
AWS_ACCESS_KEY_ID=AKIAxxxxxxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
AWS_DEFAULT_REGION=us-east-1
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

## Project structure

- `app.py`: Streamlit UI + orchestration (upload → preprocessing → inference → mixing)
- `src/aws.py`: S3/SQS I/O (upload/download, queue messages, local cache)
- `src/audio.py`: Mixing / audio rendering
- `src/config.py`: Defaults and env-var configuration
- `voice_previews/`: Local MP3 previews for available voice models


