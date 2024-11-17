# ANTHROPIC Chat Completion

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



## Streaming Responses

The **streaming** option allows you to receive messages incrementally as they are generated by the model. Streaming responses are delivered as **Server-Sent Events (SSEs)**, which allow for low-latency updates during the response generation.

---

## Example cURL Request with Streaming

This example sends a single message and streams the response incrementally.

```bash
curl https://api.anthropic.com/v1/messages \
     --header "anthropic-version: 2023-06-01" \
     --header "content-type: application/json" \
     --header "x-api-key: $ANTHROPIC_API_KEY" \
     --data \
'{
  "model": "claude-3-5-sonnet-20241022",
  "messages": [{"role": "user", "content": "Hello"}],
  "max_tokens": 256,
  "stream": true
}'
```

---

## Example Streaming Response

The response is streamed as **events**. Each event provides a specific type of update, such as the start of the message, incremental text updates, or the completion of the message.

```plaintext
event: message_start
data: {"type": "message_start", "message": {"id": "msg_1nZdL29xx5MUA1yADyHTEsnR8uuvGzszyY", "type": "message", "role": "assistant", "content": [], "model": "claude-3-5-sonnet-20241022", "stop_reason": null, "stop_sequence": null, "usage": {"input_tokens": 25, "output_tokens": 1}}}

event: content_block_start
data: {"type": "content_block_start", "index": 0, "content_block": {"type": "text", "text": ""}}

event: ping
data: {"type": "ping"}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "Hello"}}

event: content_block_delta
data: {"type": "content_block_delta", "index": 0, "delta": {"type": "text_delta", "text": "!"}}

event: content_block_stop
data: {"type": "content_block_stop", "index": 0}

event: message_delta
data: {"type": "message_delta", "delta": {"stop_reason": "end_turn", "stop_sequence":null}, "usage": {"output_tokens": 15}}

event: message_stop
data: {"type": "message_stop"}
```

---

## Event Types in Streaming Response

| Event Name               | Description                                                                                       |
|--------------------------|---------------------------------------------------------------------------------------------------|
| **message_start**        | Indicates the beginning of the message response. Contains initial metadata like the message ID.   |
| **content_block_start**  | Marks the start of a content block (e.g., text or image).                                         |
| **ping**                 | A periodic ping sent to maintain the connection.                                                 |
| **content_block_delta**  | Contains incremental updates (deltas) to the content block.                                       |
| **content_block_stop**   | Indicates the end of a content block.                                                            |
| **message_delta**        | Provides updates to the message's metadata (e.g., stop reason, token usage).                     |
| **message_stop**         | Marks the end of the streaming response.                                                         |

---

## Notes

1. **Streaming Activation**:
   - Set `"stream": true` in the request body to enable streaming responses.

2. **Latency**:
   - Streaming is ideal for low-latency applications, as content is delivered as soon as it is generated.

3. **Connection Handling**:
   - Streaming responses are sent as a continuous stream. Ensure your client can process **Server-Sent Events (SSEs)**.

4. **Event Order**:
   - Events are delivered in the following order: 
     - `message_start` → `content_block_start` → `content_block_delta` → `content_block_stop` → `message_delta` → `message_stop`.

5. **Usage Statistics**:
   - Token usage (`input_tokens` and `output_tokens`) is updated dynamically in the `message_delta` and `message_stop` events.

6. **Partial Responses**:
   - You can use the `content_block_delta` events to display partial responses to users in real-time.

