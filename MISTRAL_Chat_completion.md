# MISTRAL Chat Completion

The **Chat Completion API** generates conversational responses based on a sequence of input messages. It provides a wide range of customization options to control the behavior and creativity of the responses.

---

## Authorization

**HTTP Authorization Scheme**: `bearer`  
**Header**:  
```
Authorization: Bearer <API_KEY>
```

---

## Endpoint
```
POST https://api.mistral.ai/v1/chat/completions
```

---

## Request Body Schema

The request body must be sent in **JSON format** with the following fields:

### Required Parameters

| Parameter        | Type                      | Description                                                                                                                  |
|------------------|---------------------------|------------------------------------------------------------------------------------------------------------------------------|
| **model**        | string or null            | The model ID to use. Use the **List Available Models API** to view all models or refer to the model overview documentation.   |
| **messages**     | array of objects          | A list of input messages, each with a `role` and `content`. Represents the conversation history or single-turn input.         |

### Optional Parameters

| Parameter           | Type                    | Default  | Description                                                                                                                      |
|---------------------|-------------------------|----------|----------------------------------------------------------------------------------------------------------------------------------|
| **temperature**     | number or null          | Varies   | Controls randomness. Recommended range: `0.0` (deterministic) to `0.7` (creative). Avoid altering with `top_p` simultaneously.  |
| **top_p**           | number                  | 1        | Nucleus sampling. Considers tokens within the top `p` probability mass. Use one of `temperature` or `top_p`, not both.          |
| **max_tokens**      | integer or null         | N/A      | Maximum tokens for the completion. Prompt tokens + `max_tokens` must not exceed the model's context length.                     |
| **stream**          | boolean                 | false    | Whether to stream the response incrementally using **Server-Sent Events (SSEs)**.                                               |
| **stop**            | string or array         | N/A      | Stop generation when the specified token(s) are encountered.                                                                    |
| **random_seed**     | integer or null         | N/A      | A seed value for random sampling, ensuring deterministic outputs across calls when set.                                         |
| **response_format** | object                  | `{}`     | The format for the response content.                                                                                            |
| **tools**           | array of objects or null| null     | A list of tools the model may invoke during the conversation.                                                                   |
| **tool_choice**     | string                  | "auto"   | Specifies how the model should use tools: `"auto"`, `"any"`, or a specific tool.                                                |
| **presence_penalty**| number                  | 0        | Penalizes the model for repeating words or phrases, encouraging diversity. Range: `-2.0` to `2.0`.                              |
| **frequency_penalty**| number                 | 0        | Penalizes the model for frequently repeating words based on frequency. Range: `-2.0` to `2.0`.                                   |
| **n**               | integer or null         | 1        | Number of completions to return. Input tokens are only billed once.                                                             |
| **safe_prompt**     | boolean                 | false    | Injects a safety prompt at the start of the conversation to ensure a safer response.                                            |

---

## Example Request

### Non-Streaming Request

```bash
curl --location "https://api.mistral.ai/v1/chat/completions" \
     --header 'Content-Type: application/json' \
     --header 'Accept: application/json' \
     --header "Authorization: Bearer $MISTRAL_API_KEY" \
     --data '{
        "model": "mistral-large-latest",
        "messages": [
          {
            "role": "user",
            "content": "Who is the best French painter? Answer in one short sentence."
          }
        ],
        "temperature": 0.5,
        "top_p": 1,
        "max_tokens": 50,
        "stream": false,
        "presence_penalty": 0,
        "frequency_penalty": 0
      }'
```

### Streaming Request

```bash
curl --location "https://api.mistral.ai/v1/chat/completions" \
     --header 'Content-Type: application/json' \
     --header 'Accept: application/json' \
     --header "Authorization: Bearer $MISTRAL_API_KEY" \
     --data '{
        "model": "mistral-large-latest",
        "messages": [
          {
            "role": "user",
            "content": "What is the best French cheese?"
          }
        ],
        "temperature": 0.7,
        "top_p": 1,
        "max_tokens": 100,
        "stream": true
      }'
```

---

## Example Responses

### Non-Streaming Response
```json
{
  "id": "cmpl-e5cc70bb28c444948073e77776eb30ef",
  "object": "chat.completion",
  "model": "mistral-large-latest",
  "usage": {
    "prompt_tokens": 16,
    "completion_tokens": 34,
    "total_tokens": 50
  },
  "created": 1702256327,
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "The best French painter is Claude Monet, a pioneer of Impressionism.",
        "tool_calls": {}
      },
      "finish_reason": "stop"
    }
  ]
}
```

### Streaming Response
```plaintext
event: message_start
data: {"type": "message_start", "message": {"id": "msg_1nZdL29xx5MUA1yADyHTEsnR8uuvGzszyY", "type": "message", "role": "assistant", "content": [], "model": "mistral-large-latest", "stop_reason": null, "stop_sequence": null, "usage": {"input_tokens": 25, "output_tokens": 1}}}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "Hello"}}

event: message_delta
data: {"type": "message_delta", "delta": {"stop_reason": "end_turn"}, "usage": {"output_tokens": 15}}

event: message_stop
data: {"type": "message_stop"}
```

---

## Responses

### Success Response (200)
| Field              | Type          | Description                                                                 |
|--------------------|---------------|-----------------------------------------------------------------------------|
| **id**             | string        | Unique identifier for the chat completion.                                 |
| **object**         | string        | Always `"chat.completion"`.                                                |
| **model**          | string        | The model used to generate the response.                                   |
| **usage**          | object        | Token usage details: `prompt_tokens`, `completion_tokens`, and `total_tokens`. |
| **created**        | integer       | Unix timestamp of when the response was generated.                         |
| **choices**        | array         | List of completions, each containing a `message`.                          |

### Error Response (422)
If validation fails, the API will return an error response.

| Field              | Type          | Description                                                                 |
|--------------------|---------------|-----------------------------------------------------------------------------|
| **type**           | string        | The type of error (e.g., `"validation_error"`).                             |
| **message**        | string        | A detailed error message.                                                  |

**Example:**
```json
{
  "type": "validation_error",
  "message": "Invalid model ID."
}
```

---

## Notes

1. **Model Compatibility**: Ensure that the `model` supports the desired parameters (e.g., `temperature`, `streaming`).
2. **Token Limits**: Total tokens (`prompt_tokens` + `max_tokens`) must not exceed the model's context length.
3. **Stop Tokens**: Use the `stop` parameter to define custom stop sequences for finer control.
4. **Randomness**: Adjust `temperature` or `top_p` to modify the creativity and determinism of the output.
5. **Streaming**: Enable `stream: true` for real-time token delivery.

