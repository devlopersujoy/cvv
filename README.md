# Voice to text
A Python/Flask web app for transcribing audio recordings, video uploads, and public YouTube videos. Select Auto for the multilingual Whisper model or choose a language manually. Bengali is available as a manual choice. Download the original-language transcript as UTF-8 text.

## Render-এ চালু করুন
1. ZIP খুলে সব ফাইল একটি GitHub repository-র মূল ফোল্ডারে আপলোড করুন। সেখানে render.yaml এবং Dockerfile পাশাপাশি থাকবে।
2. Render Dashboard খুলে New → Blueprint নির্বাচন করুন এবং ওই repository সংযুক্ত করুন।
3. Free web service দেখে Deploy Blueprint চাপুন। প্রথম build-এ whisper.cpp কম্পাইল হবে ও মডেল ডাউনলোড হবে।
4. Deploy সফল হলে Render-এর দেওয়া URL খুলুন। ছোট একটি বাংলা ভয়েস ফাইল দিয়ে শুরু করুন।

## Local Docker
Run these commands from the extracted folder.

    docker build -t voice-to-text .
    docker run --rm -p 10000:10000 voice-to-text

Open http://localhost:10000. No transcription API key is needed. Docker build needs internet to install dependencies and fetch the multilingual Tiny Q5_1 model. YouTube retrieval also needs internet.

## Operation
One active clip per instance. Keep the tab open while processing. Maximum 48 MiB and 10 minutes per clip. The tiny model is chosen for CPU and low memory. Accuracy varies by language, accent, noise and recording quality. The model covers a defined set of languages. All 99 supported languages are selectable.

The server deletes uploaded and extracted media when the job finishes. Results are temporarily held in process memory, available for up to an hour or until restart. YouTube can restrict cloud-hosted requests. If retrieval fails, upload an audio or video copy you can access. Only individual videos are accepted. Live streams and playlists are excluded. Rebuild periodically to pick up yt-dlp extractor fixes.

## HTTP API
Replace BASE with your deployment URL. POST a link as JSON (YouTube, public audio, or public video) or send one local file as multipart. Set language to auto or any of the 99 model codes returned by GET /api/v1/languages. This also lists the languages available in the web menu.

    curl -X POST "$BASE/api/v1/transcriptions" -H 'Content-Type: application/json' -d '{"url":"https://example.org/voice.ogg","language":"bn"}'
    curl -X POST "$BASE/api/v1/transcriptions" -F 'file=@video.mp4' -F 'language=auto'

A successful submission returns HTTP 202 with id and status_url. Poll GET on BASE plus status_url every two seconds. The final JSON has state "Complete" and text. A failed job contains error. HTTP 400 indicates invalid input, 413 an oversized upload, 429 a busy worker, and 404 an expired or unknown job. Anyone with the service URL can submit jobs. Keep the job URL private. Direct links must serve downloadable media over public HTTP or HTTPS. Redirects are supported. HTML sharing pages and authenticated media hosts usually require a file upload instead.
