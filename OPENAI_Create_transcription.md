# Create Transcription API Documentation

The **Create Transcription API** transcribes audio files into text in the input language.

---

## Endpoint
```
POST https://api.openai.com/v1/audio/transcriptions
```

---

## Request Body

### Parameters

| Name                    | Type          | Required | Description                                                                                              |
|-------------------------|---------------|----------|----------------------------------------------------------------------------------------------------------|
| **file**                | file          | Required | The audio file object (not the file name) to transcribe. Supported formats: `flac`, `mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `ogg`, `wav`, or `webm`. |
| **model**               | string        | Required | ID of the model to use. Currently, only `whisper-1` (powered by the Whisper V2 model) is available.      |
| **language**            | string        | Optional | The language of the input audio in ISO-639-1 format. Supplying this can improve accuracy and reduce latency. |
| **prompt**              | string        | Optional | A text prompt to guide the model's style or to continue a previous audio segment. The prompt must match the audio language. |
| **response_format**     | string        | Optional | The output format. Supported formats: `json`, `text`, `srt`, `verbose_json`, or `vtt`. Defaults to `json`. |
| **temperature**         | number        | Optional | Sampling temperature (0 to 1). Lower values make the output more deterministic, while higher values add randomness. Defaults to `0`. |
| **timestamp_granularities[]** | array | Optional | Granularities for timestamps. Used only with `response_format="verbose_json"`. Options: `word` or `segment`. Defaults to `segment`. |

---

## Returns

- **Transcription Object**: Contains the transcribed text.
- **Verbose Transcription Object**: Includes additional metadata like timestamps and tokens.

---

## Example cURL Requests

### 1. Basic Transcription

This example transcribes an audio file (`audio.mp3`) using the `whisper-1` model and outputs the text in JSON format.

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F file="@/path/to/file/audio.mp3" \
  -F model="whisper-1"
```

**Response:**
```json
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger. This is a place where you can get to do that."
}
```

---

### 2. Transcription with Word Timestamps

This example transcribes `audio.mp3` and includes word-level timestamps in the response.

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F file="@/path/to/file/audio.mp3" \
  -F "timestamp_granularities[]=word" \
  -F model="whisper-1" \
  -F response_format="verbose_json"
```

**Response:**
```json
{
  "task": "transcribe",
  "language": "english",
  "duration": 8.47,
  "text": "The beach was a popular spot on a hot summer day. People were swimming in the ocean, building sandcastles, and playing beach volleyball.",
  "words": [
    {
      "word": "The",
      "start": 0.0,
      "end": 0.24
    },
    ...
    {
      "word": "volleyball",
      "start": 7.4,
      "end": 7.9
    }
  ]
}
```

---

### 3. Transcription with Segment Timestamps

This example transcribes `audio.mp3` and includes segment-level timestamps in the response.

```bash
curl https://api.openai.com/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F file="@/path/to/file/audio.mp3" \
  -F "timestamp_granularities[]=segment" \
  -F model="whisper-1" \
  -F response_format="verbose_json"
```

**Response:**
```json
{
  "task": "transcribe",
  "language": "english",
  "duration": 8.47,
  "text": "The beach was a popular spot on a hot summer day. People were swimming in the ocean, building sandcastles, and playing beach volleyball.",
  "segments": [
    {
      "id": 0,
      "seek": 0,
      "start": 0.0,
      "end": 3.32,
      "text": "The beach was a popular spot on a hot summer day.",
      "tokens": [50364, 440, 7534, 390, 257, 3743, 4008, 322, 257, 2368, 4266, 786, 13, 50530],
      "temperature": 0.0,
      "avg_logprob": -0.286,
      "compression_ratio": 1.236,
      "no_speech_prob": 0.0099
    },
    ...
  ]
}
```

---

## Notes

- **Supported Audio Formats**: Ensure your audio file is in one of the supported formats: `flac`, `mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `ogg`, `wav`, or `webm`.
- **API Key**: Replace `$OPENAI_API_KEY` with your actual API key.
- **Language**: Specifying the `language` parameter in ISO-639-1 format can improve the transcription accuracy.
- **Timestamps**: Use `timestamp_granularities[]` for more detailed timestamp information at the word or segment level.
- **Output Format**: Use the `response_format` parameter to customize the output format (`json`, `text`, `srt`, `verbose_json`, `vtt`).
