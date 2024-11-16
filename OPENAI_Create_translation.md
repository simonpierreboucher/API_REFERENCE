# Create Translation API Documentation

The **Create Translation API** translates audio files into English.

---

## Endpoint
```
POST https://api.openai.com/v1/audio/translations
```

---

## Request Body

### Parameters

| Name                 | Type   | Required | Description                                                                                             |
|----------------------|--------|----------|---------------------------------------------------------------------------------------------------------|
| **file**             | file   | Required | The audio file object (not the file name) to translate. Supported formats: `flac`, `mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `ogg`, `wav`, or `webm`. |
| **model**            | string | Required | ID of the model to use. Currently, only `whisper-1` (powered by the Whisper V2 model) is available.     |
| **prompt**           | string | Optional | A text prompt to guide the model's style or continue a previous audio segment. The prompt must be in English. |
| **response_format**  | string | Optional | The output format. Supported formats: `json`, `text`, `srt`, `verbose_json`, or `vtt`. Defaults to `json`. |
| **temperature**      | number | Optional | Sampling temperature (0 to 1). Lower values make the output more deterministic, while higher values add randomness. Defaults to `0`. |

---

## Returns

- **Translated Text**: The transcription of the audio translated into English.

---

## Example cURL Request

This example translates an audio file (`german.m4a`) into English using the `whisper-1` model.

```bash
curl https://api.openai.com/v1/audio/translations \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: multipart/form-data" \
  -F file="@/path/to/file/german.m4a" \
  -F model="whisper-1"
```

**Response:**
```json
{
  "text": "Hello, my name is Wolfgang and I come from Germany. Where are you heading today?"
}
```

---

## Notes

- **Supported Audio Formats**: Ensure your audio file is in one of the supported formats: `flac`, `mp3`, `mp4`, `mpeg`, `mpga`, `m4a`, `ogg`, `wav`, or `webm`.
- **API Key**: Replace `$OPENAI_API_KEY` with your actual API key.
- **Prompt**: Use the `prompt` parameter to influence the translation style or continue from a prior segment.
- **Output Format**: The `response_format` parameter allows you to select the desired output format (`json`, `text`, `srt`, `verbose_json`, or `vtt`).
- **Deterministic Results**: Set `temperature` to `0` for consistent translations or increase for more varied output.

