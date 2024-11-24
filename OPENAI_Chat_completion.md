# OPENAI Chat Completion

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

### Example 4: Using Tools in Chat Completion

#### Request:
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "user",
        "content": "What'\''s the weather like in Boston today?"
      }
    ],
    "tools": [
      {
        "type": "function",
        "function": {
          "name": "get_current_weather",
          "description": "Get the current weather in a given location",
          "parameters": {
            "type": "object",
            "properties": {
              "location": {
                "type": "string",
                "description": "The city and state, e.g. San Francisco, CA"
              },
              "unit": {
                "type": "string",
                "enum": ["celsius", "fahrenheit"]
              }
            },
            "required": ["location"]
          }
        }
      }
    ],
    "tool_choice": "auto"
  }'
```

#### Response:
```json
{
  "id": "chatcmpl-abc123",
  "object": "chat.completion",
  "created": 1699896916,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": null,
        "tool_calls": [
          {
            "id": "call_abc123",
            "type": "function",
            "function": {
              "name": "get_current_weather",
              "arguments": "{\n\"location\": \"Boston, MA\"\n}"
            }
          }
        ]
      },
      "logprobs": null,
      "finish_reason": "tool_calls"
    }
  ],
  "usage": {
    "prompt_tokens": 82,
    "completion_tokens": 17,
    "total_tokens": 99,
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  }
}
```

---

### Example 5: Including Log Probabilities

#### Request:
```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4o",
    "messages": [
      {
        "role": "user",
        "content": "Hello!"
      }
    ],
    "logprobs": true,
    "top_logprobs": 2
  }'
```

#### Response:
```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1702685778,
  "model": "gpt-4o-mini",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "Hello! How can I assist you today?"
      },
      "logprobs": {
        "content": [
          {
            "token": "Hello",
            "logprob": -0.31725305,
            "bytes": [72, 101, 108, 108, 111],
            "top_logprobs": [
              {
                "token": "Hello",
                "logprob": -0.31725305,
                "bytes": [72, 101, 108, 108, 111]
              },
              {
                "token": "Hi",
                "logprob": -1.3190403,
                "bytes": [72, 105]
              }
            ]
          },
          {
            "token": "!",
            "logprob": -0.02380986,
            "bytes": [33],
            "top_logprobs": [
              {
                "token": "!",
                "logprob": -0.02380986,
                "bytes": [33]
              },
              {
                "token": " there",
                "logprob": -3.787621,
                "bytes": [32, 116, 104, 101, 114, 101]
              }
            ]
          },
          {
            "token": " How",
            "logprob": -0.000054669687,
            "bytes": [32, 72, 111, 119],
            "top_logprobs": [
              {
                "token": " How",
                "logprob": -0.000054669687,
                "bytes": [32, 72, 111, 119]
              },
              {
                "token": "<|end|>",
                "logprob": -10.953937,
                "bytes": null
              }
            ]
          },
          {
            "token": " can",
            "logprob": -0.015801601,
            "bytes": [32, 99, 97, 110],
            "top_logprobs": [
              {
                "token": " can",
                "logprob": -0.015801601,
                "bytes": [32, 99, 97, 110]
              },
              {
                "token": " may",
                "logprob": -4.161023,
                "bytes": [32, 109, 97, 121]
              }
            ]
          },
          {
            "token": " I",
            "logprob": -3.7697225e-6,
            "bytes": [32, 73],
            "top_logprobs": [
              {
                "token": " I",
                "logprob": -3.7697225e-6,
                "bytes": [32, 73]
              },
              {
                "token": " assist",
                "logprob": -13.596657,
                "bytes": [32, 97, 115, 115, 105, 115, 116]
              }
            ]
          },
          {
            "token": " assist",
            "logprob": -0.04571125,
            "bytes": [32, 97, 115, 115, 105, 115, 116],
            "top_logprobs": [
              {
                "token": " assist",
                "logprob": -0.04571125,
                "bytes": [32, 97, 115, 115, 105, 115, 116]
              },
              {
                "token": " help",
                "logprob": -3.1089056,
                "bytes": [32, 104, 101, 108, 112]
              }
            ]
          },
          {
            "token": " you",
            "logprob": -5.4385737e-6,
            "bytes": [32, 121, 111, 117],
            "top_logprobs": [
              {
                "token": " you",
                "logprob": -5.4385737e-6,
                "bytes": [32, 121, 111, 117]
              },
              {
                "token": " today",
                "logprob": -12.807695,
                "bytes": [32, 116, 111, 100, 97, 121]
              }
            ]
          },
          {
            "token": " today",
            "logprob": -0.0040071653,
            "bytes": [32, 116, 111, 100, 97, 121],
            "top_logprobs": [
              {
                "token": " today",
                "logprob": -0.0040071653,
                "bytes": [32, 116, 111, 100, 97, 121]
              },
              {
                "token": "?",
                "logprob": -5.5247097,
                "bytes": [63]
              }
            ]
          },
          {
            "token": "?",
            "logprob": -0.0008108172,
            "bytes": [63],
            "top_logprobs": [
              {
                "token": "?",
                "logprob": -0.0008108172,
                "bytes": [63]
              },
              {
                "token": "?\n",
                "logprob": -7.184561,
                "bytes": [63, 10]
              }
            ]
          }
        ]
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 9,
    "completion_tokens": 9,
    "total_tokens": 18,
    "completion_tokens_details": {
      "reasoning_tokens": 0,
      "accepted_prediction_tokens": 0,
      "rejected_prediction_tokens": 0
    }
  },
  "system_fingerprint": null
}
```

---

---
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

