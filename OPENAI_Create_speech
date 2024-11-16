
# Create Speech API Documentation

The **Create Speech API** generates audio from input text.

## Endpoint
```
POST https://api.openai.com/v1/audio/speech
```

## Request Body

### Parameters

| Name             | Type    | Required | Description                                                                                              |
|------------------|---------|----------|----------------------------------------------------------------------------------------------------------|
| **model**        | string  | Required | One of the available TTS models: `tts-1` or `tts-1-hd`.                                                 |
| **input**        | string  | Required | The text to generate audio for. The maximum length is 4096 characters.                                  |
| **voice**        | string  | Required | The voice to use for generating audio. Supported voices: `alloy`, `echo`, `fable`, `onyx`, `nova`, and `shimmer`. |
| **response_format** | string  | Optional | The format of the output audio. Supported formats: `mp3`, `opus`, `aac`, `flac`, `wav`, and `pcm`. Defaults to `mp3`. |
| **speed**        | number  | Optional | The speed of the generated audio. Values range from `0.25` to `4.0`. Defaults to `1.0`.                 |

### Returns

- **Audio file content**: The generated audio file in the specified format.

---

## Example cURL Request

This example generates audio for the text *"The quick brown fox jumped over the lazy dog."* using the `tts-1` model and the `alloy` voice. The output file is saved as `speech.mp3`.

```bash
curl https://api.openai.com/v1/audio/speech \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "tts-1",
    "input": "The quick brown fox jumped over the lazy dog.",
    "voice": "alloy"
  }' \
  --output speech.mp3
```

### Response Format

The response contains the audio file content. If the `--output` flag is used, the file will be saved locally (e.g., `speech.mp3`).

---

## Supported Voices

- **alloy**
- **echo**
- **fable**
- **onyx**
- **nova**
- **shimmer**

For a preview of the voices, refer to the [Text-to-Speech Guide](#).

---

## Notes

- Ensure that the `OPENAI_API_KEY` environment variable contains your API key.
- The text length should not exceed 4096 characters.
- Use the `response_format` parameter to specify audio file types if needed.
