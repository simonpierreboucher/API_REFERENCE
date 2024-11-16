# Messages API Documentation

The **Messages API** facilitates conversational interactions by generating model responses based on structured input messages. It supports both single-turn and multi-turn stateless conversations.

---

## Endpoint
```
POST https://api.anthropic.com/v1/messages
```

---

## Headers

| Header               | Type    | Required | Description                                                                 |
|----------------------|---------|----------|-----------------------------------------------------------------------------|
| **x-api-key**        | string  | Required | Your unique API key for authentication. Obtain this key from the console.  |
| **anthropic-version**| string  | Required | The API version to use (e.g., `2023-06-01`).                               |
| **content-type**     | string  | Required | Must be `application/json`.                                                |
| **anthropic-beta**   | string[]| Optional | Specify one or more beta versions to use (e.g., `beta1,beta2`).            |

---

## Request Body

### Parameters

| Name             | Type             | Required | Description                                                                                                                                                          |
|------------------|------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **model**        | string           | Required | The model to use (e.g., `claude-3-5-sonnet-20241022`).                                                                                                               |
| **messages**     | object[]         | Required | A list of input messages. Each message must include a `role` (user or assistant) and `content`.                                                                     |
| **max_tokens**   | integer          | Required | The maximum number of tokens to generate before stopping.                                                                                                           |
| **system**       | string           | Optional | A system prompt providing context or instructions for the conversation.                                                                                             |
| **stop_sequences**| string[]        | Optional | Custom sequences that will cause the model to stop generating.                                                                                                      |
| **temperature**  | number           | Optional | Defaults to `1.0`. Controls the randomness of the output. Use values closer to `0.0` for deterministic outputs and higher values for creative tasks.                |
| **stream**       | boolean          | Optional | Enables incremental streaming of responses using server-sent events.                                                                                                |
| **metadata**     | object           | Optional | Metadata describing the request (e.g., tags or additional identifiers).                                                                                             |
| **tools**        | object[]         | Optional | A list of tools the model may use. Each tool must include a `name`, `description`, and `input_schema`.                                                              |
| **tool_choice**  | object           | Optional | Specifies how the model should use the provided tools. Options: `auto`, `any`, or specify a specific tool.                                                          |
| **top_k**        | integer          | Optional | Limits token sampling to the top K most probable tokens.                                                                                                            |
| **top_p**        | number           | Optional | Implements nucleus sampling. Limits token sampling to a cumulative probability mass of `top_p`.                                                                    |

---

## Response

The API returns either a **message object** or an **error object**.

### Message Object Structure

| Field             | Type             | Description                                                                                                             |
|-------------------|------------------|-------------------------------------------------------------------------------------------------------------------------|
| **id**            | string           | A unique identifier for the message.                                                                                   |
| **type**          | string           | Always `message`.                                                                                                      |
| **role**          | string           | The role of the generated message. Always `assistant`.                                                                 |
| **content**       | object[]         | An array of content blocks generated by the model. Each block has a `type` (e.g., `text`) and the corresponding `text`. |
| **model**         | string           | The model that processed the request.                                                                                  |
| **stop_reason**   | string or null   | The reason for stopping. Possible values: `end_turn`, `max_tokens`, `stop_sequence`, or `tool_use`.                     |
| **stop_sequence** | string or null   | The custom stop sequence that triggered stopping, if applicable.                                                       |
| **usage**         | object           | Token usage information. Includes `input_tokens` and `output_tokens`.                                                  |

#### Token Usage

| Field            | Type    | Description                              |
|------------------|---------|------------------------------------------|
| **input_tokens** | integer | Number of tokens in the input messages.  |
| **output_tokens**| integer | Number of tokens in the generated output.|

---

### Error Object Structure

| Field            | Type    | Description                     |
|------------------|---------|---------------------------------|
| **type**         | string  | Always `error`.                |
| **error**        | object  | Details of the error.          |
| **error.type**   | string  | The type of error encountered. |
| **error.message**| string  | A detailed error message.      |

---

## Example cURL Request

### 1. Basic Conversation

```bash
curl https://api.anthropic.com/v1/messages \
     --header "x-api-key: $ANTHROPIC_API_KEY" \
     --header "anthropic-version: 2023-06-01" \
     --header "content-type: application/json" \
     --data \
'{
    "model": "claude-3-5-sonnet-20241022",
    "max_tokens": 1024,
    "messages": [
        {"role": "user", "content": "Hello, world"}
    ]
}'
```

**Response:**
```json
{
  "content": [
    {
      "text": "Hi! My name is Claude.",
      "type": "text"
    }
  ],
  "id": "msg_013Zva2CMHLNnXjNJJKqJ2EF",
  "model": "claude-3-5-sonnet-20241022",
  "role": "assistant",
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "type": "message",
  "usage": {
    "input_tokens": 2095,
    "output_tokens": 503
  }
}
```

### 2. Error Handling

**Response Example:**
```json
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "Invalid model name"
  }
}
```

---

## Notes

1. **Conversation Context**:
   - Specify prior conversational turns in the `messages` parameter for multi-turn interactions.
   - Consecutive `user` or `assistant` turns are combined automatically.

2. **Input Formats**:
   - Input messages can include both text and images (starting with Claude 3 models).
   - Use `base64` encoding for images.

3. **System Prompts**:
   - Use the `system` parameter to provide context or instructions, such as defining the assistant's role or task.

4. **Tool Integration**:
   - Define tools the model can invoke for advanced workflows (e.g., JSON-based tools for fetching external data).

5. **Token Limits**:
   - Ensure your `max_tokens` value does not exceed the model's maximum token capacity.

6. **Streaming**:
   - Set `stream: true` to receive responses incrementally as they are generated.

---
