# Create Chat Completion API Documentation

The **Create Chat Completion API** generates responses for a given conversation using a specified model.

---

## Endpoint
```
POST https://api.openai.com/v1/chat/completions
```

---

## Request Body

### Parameters

| Name                     | Type            | Required | Description                                                                                              |
|--------------------------|-----------------|----------|----------------------------------------------------------------------------------------------------------|
| **messages**             | array           | Required | A list of messages in the conversation so far. Supported types include `text`, `images`, and `audio`. |
| **model**                | string          | Required | ID of the model to use. See the model endpoint compatibility table for available models.                |
| **store**                | boolean/null    | Optional | Defaults to `false`. Whether to store the output of this request for use in model distillation or evals. |
| **metadata**             | object/null     | Optional | Developer-defined tags/values for filtering completions in the dashboard.                               |
| **frequency_penalty**    | number/null     | Optional | Defaults to `0`. Penalizes new tokens based on their frequency. Range: `-2.0` to `2.0`.                 |
| **logit_bias**           | map             | Optional | Modify the likelihood of specific tokens. Maps token IDs to bias values from `-100` to `100`.           |
| **logprobs**             | boolean/null    | Optional | Defaults to `false`. Returns log probabilities of output tokens if set to `true`.                       |
| **top_logprobs**         | integer/null    | Optional | Specifies the number of top tokens with log probabilities to return. Requires `logprobs: true`.         |
| **max_completion_tokens**| integer/null    | Optional | Upper bound for tokens generated in the response. Replaces the deprecated `max_tokens`.                 |
| **n**                    | integer/null    | Optional | Defaults to `1`. Specifies the number of completion choices to generate for each input message.         |
| **modalities**           | array/null      | Optional | Output types to generate (e.g., `["text"]`, or `["text", "audio"]` for multi-modal responses).           |
| **prediction**           | object          | Optional | Configuration for predicted outputs to improve response times for known content.                        |
| **audio**                | object/null     | Optional | Parameters for audio output. Required for `modalities: ["audio"]`.                                      |
| **presence_penalty**     | number/null     | Optional | Defaults to `0`. Penalizes tokens based on their presence. Range: `-2.0` to `2.0`.                      |
| **response_format**      | object          | Optional | Specifies the output format (e.g., `json_schema`, `json_object`).                                        |
| **seed**                 | integer/null    | Optional | Ensures determinism in sampling. Responses may vary slightly but aim to be consistent.                  |
| **service_tier**         | string/null     | Optional | Defaults to `auto`. Specifies the service tier for processing requests (e.g., `scale` or `default`).    |
| **stop**                 | string/array/null | Optional | Defaults to `null`. Up to 4 sequences where the API will stop generating further tokens.                 |
| **stream**               | boolean/null    | Optional | Defaults to `false`. Enables token streaming as server-sent events (`data: [DONE]`).                     |
| **temperature**          | number/null     | Optional | Defaults to `1`. Sampling temperature (0-2). Higher values increase randomness.                         |
| **top_p**                | number/null     | Optional | Defaults to `1`. Uses nucleus sampling. Tokens within the `top_p` probability mass are considered.      |
| **tools**                | array           | Optional | List of tools (e.g., functions) the model can call. Max of 128 tools.                                   |
| **tool_choice**          | string/object   | Optional | Controls how tools are used (`none`, `auto`, or specify a tool).                                         |
| **parallel_tool_calls**  | boolean         | Optional | Defaults to `true`. Enables parallel function calling during tool use.                                  |
| **user**                 | string          | Optional | Unique identifier for monitoring end-user behavior.                                                     |

---

## Returns

- **Chat Completion Object**: Contains the model's response and metadata.
- **Streamed Chat Completion**: A sequence of token chunks when `stream: true`.

---

## Example cURL Requests

### 1. Basic Chat Completion
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ]
  }'
```

**Response:**
```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-4o-mini",
  "system_fingerprint": "fp_44709d6fcb",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "Hello there, how may I assist you today?"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 12,
    "total_tokens": 21
  }
}
```

---

### 2. Multi-Modal Chat Completion (Text and Image)
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "user",
        "content": [
          {
            "type": "text",
            "text": "What's in this image?"
          },
          {
            "type": "image_url",
            "image_url": {
              "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg"
            }
          }
        ]
      }
    ],
    "max_completion_tokens": 300
  }'
```

**Response:**
```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677652288,
  "model": "gpt-4o-mini",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "This image shows a wooden boardwalk extending through a lush green marshland."
    },
    "finish_reason": "stop"
  }]
}
```

---

### 3. Streamed Chat Completion
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "system",
        "content": "You are a helpful assistant."
      },
      {
        "role": "user",
        "content": "Hello!"
      }
    ],
    "stream": true
  }'
```

**Response:** The response will be a sequence of chunks:
```json
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "choices":[{"delta":{"content":"Hello"}}]}
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "choices":[{"delta":{"content":" there,"}}]}
...
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "choices":[{"delta":{"content":" how may I assist you today?"}}]}
{"id":"chatcmpl-123","object":"chat.completion.chunk","created":1694268190,"model":"gpt-4o-mini", "choices":[{"finish_reason":"stop"}]}
```

---
