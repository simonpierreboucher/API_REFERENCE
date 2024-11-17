# Documentation for Cohere Chat API (POST)

The Cohere Chat API allows you to generate text responses to user messages, supporting tools and system messages for advanced conversation flows. This endpoint is especially useful for creating conversational AI applications.

## Endpoint
**POST** `https://api.cohere.com/v2/chat`

### Purpose
Generates a text response to a user message, optionally streaming the response token by token.

---

## Headers

| Header Name         | Type   | Required | Description                              |
|---------------------|--------|----------|------------------------------------------|
| `X-Client-Name`     | String | Optional | Name of the project making the request. |
| `Authorization`     | String | Required | Bearer token for authentication.        |
| `accept`            | String | Required | Must be set to `application/json`.       |
| `content-type`      | String | Required | Must be set to `application/json`.       |

---

## Request

The request body expects a JSON object with the following structure:

### Required Fields

| Field           | Type   | Required | Description                                                                                     |
|------------------|--------|----------|-------------------------------------------------------------------------------------------------|
| `model`         | String | Yes      | Name of the model to use (e.g., `command-r` or `command-r-plus`).                              |
| `messages`      | Array  | Yes      | List of messages representing the conversation. See "Message Structure" below for details.     |

### Optional Fields

| Field               | Type    | Description                                                                                                         |
|---------------------|---------|---------------------------------------------------------------------------------------------------------------------|
| `stream`            | Boolean | If true, streams the response token by token.                                                                      |
| `tools`             | Array   | List of available tools (functions) the model can suggest invoking.                                                |
| `response_format`   | Object  | Ensures a specific output format, like JSON. Can include a JSON schema.                                            |
| `safety_mode`       | Enum    | Options: `CONTEXTUAL`, `STRICT`, `OFF`. Default: `CONTEXTUAL`. Determines the safety level for generated content.   |
| `max_tokens`        | Integer | Limits the number of tokens in the response.                                                                       |
| `temperature`       | Float   | Controls response randomness. Default: `0.3`. Lower = more deterministic, higher = more random.                   |
| `frequency_penalty` | Float   | Penalizes repetition of tokens. Range: 0.0 to 1.0.                                                                 |
| `presence_penalty`  | Float   | Penalizes the presence of repeated tokens. Range: 0.0 to 1.0.                                                      |
| `k`                 | Float   | Considers the top-k most likely tokens at each step. Range: 0 to 500.                                              |
| `p`                 | Float   | Ensures only tokens with total probability `p` are considered. Range: 0.01 to 0.99.                                |

---

### Message Structure

Messages must include a `role` and `content`. Roles can be `user`, `assistant`, `system`, or `tool`. Content varies by role.

#### User Message

```json
{
  "role": "user",
  "content": {
    "type": "text",
    "text": "Hello world!"
  }
}
```

#### Assistant Message

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ]
}
```

#### System Message

```json
{
  "role": "system",
  "content": {
    "type": "text",
    "text": "You are a helpful assistant."
  }
}
```

#### Tool Message

```json
{
  "role": "tool",
  "content": {
    "type": "text",
    "text": "{\"action\": \"search\", \"query\": \"weather today\"}"
  }
}
```

---

## Response

The response returns a JSON object with the following fields:

| Field            | Type   | Description                                                                                      |
|-------------------|--------|--------------------------------------------------------------------------------------------------|
| `id`             | String | Unique identifier for the response.                                                             |
| `finish_reason`  | Enum   | Reason for finishing the response. Possible values: `COMPLETE`, `MAX_TOKENS`, `STOP_SEQUENCE`.  |
| `message`        | Object | The generated message from the assistant.                                                       |
| `usage`          | Object | Details about token usage and billing.                                                          |



---

## Example cURL Command

```bash
curl --request POST \
  --url https://api.cohere.com/v2/chat \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --header "Authorization: Bearer $CO_API_KEY" \
  --data '{
    "model": "command-r-plus-08-2024",
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Hello world!"
        }
      }
    ]
  }'
```

### Example Response

```json
{
  "id": "c14c80c3-18eb-4519-9460-6c92edd8cfb4",
  "finish_reason": "COMPLETE",
  "message": {
    "role": "assistant",
    "content": [
      {
        "type": "text",
        "text": "Hello! How can I assist you today?"
      }
    ]
  },
  "usage": {
    "billed_units": {
      "input_tokens": 5,
      "output_tokens": 418
    },
    "tokens": {
      "input_tokens": 71,
      "output_tokens": 418
    }
  }
}
```


# Documentation for Chat API with Streaming (POST)

The **Chat API with Streaming** endpoint allows you to generate text responses to user messages in real time, token by token. This feature is ideal for building interactive and responsive applications.

---

## Endpoint

**POST** `https://api.cohere.com/v2/chat`

---

## Purpose

Generate a streamed token-by-token response to user input, providing real-time conversational feedback.

---

## Headers

| Header Name         | Type   | Required | Description                              |
|---------------------|--------|----------|------------------------------------------|
| `Authorization`     | String | Required | Bearer token for authentication.        |
| `accept`            | String | Required | Must be set to `application/json`.       |
| `content-type`      | String | Required | Must be set to `application/json`.       |
| `X-Client-Name`     | String | Optional | Name of the project making the request. |

---

## Request Body

The request expects a JSON object with the following structure:

### Required Fields

| Field           | Type   | Description                                                                                     |
|------------------|--------|-------------------------------------------------------------------------------------------------|
| `stream`        | Boolean | Set to `true` to enable streaming responses.                                                   |
| `model`         | String | Name of the Cohere model to use, such as `command-r-plus-08-2024`.                              |
| `messages`      | Array  | List of messages representing the conversation. See "Message Structure" below.                 |

### Optional Fields

| Field               | Type    | Description                                                                                                         |
|---------------------|---------|---------------------------------------------------------------------------------------------------------------------|
| `tools`             | Array   | List of tools (functions) the model can invoke.                                                                    |
| `response_format`   | Object  | Specifies the desired output format, including optional JSON schema for structure enforcement.                     |
| `safety_mode`       | Enum    | Options: `CONTEXTUAL`, `STRICT`, `OFF`. Default: `CONTEXTUAL`. Controls safety level for content generation.        |
| `max_tokens`        | Integer | Limits the maximum number of tokens generated in the response.                                                     |
| `temperature`       | Float   | Controls randomness in the output. Default: `0.3`. Lower = deterministic, higher = more creative.                  |
| `frequency_penalty` | Float   | Penalizes token repetition. Range: 0.0 to 1.0.                                                                     |
| `presence_penalty`  | Float   | Penalizes repeated token presence. Range: 0.0 to 1.0.                                                              |
| `k`                 | Float   | Restricts output to the top-k most likely tokens. Range: 0 to 500.                                                 |
| `p`                 | Float   | Restricts output to tokens within a cumulative probability `p`. Range: 0.01 to 0.99.                               |

---

### Message Structure

Messages must include a `role` and `content`. Supported roles are:
- **user**: The input message from the user.
- **assistant**: The model's response.
- **system**: Instructions or configurations for the assistant.
- **tool**: Information related to a tool invocation.

#### Example Message Formats

**User Message:**
```json
{
  "role": "user",
  "content": {
    "type": "text",
    "text": "Hello world!"
  }
}
```

**Assistant Message:**
```json
{
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I assist you today?"
    }
  ]
}
```

**System Message:**
```json
{
  "role": "system",
  "content": {
    "type": "text",
    "text": "You are a helpful assistant."
  }
}
```

---

## Server-Sent Events (Streaming)

When `stream` is set to `true`, the response is delivered as **Server-Sent Events (SSE)**, enabling real-time delivery of tokens.

### Event Types

| Event Type         | Description                                                                                   |
|---------------------|-----------------------------------------------------------------------------------------------|
| `message-start`    | Indicates the start of the assistant's response message.                                      |
| `content-start`    | Signals the beginning of a new content block.                                                 |
| `content-delta`    | Contains a chunk of text (token). Multiple deltas make up the full message.                   |
| `content-end`      | Indicates the end of a content block.                                                         |
| `message-end`      | Marks the end of the response stream, including usage statistics and finish reason.            |

---

### Example Event Stream


### Example cURL Command

```bash
curl --request POST \
  --url https://api.cohere.com/v2/chat \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --header "Authorization: Bearer $CO_API_KEY" \
  --data '{
    "stream": true,
    "model": "command-r-plus-08-2024",
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Hello world!"
        }
      }
    ]
  }'
```
---
#### Start of Message
```json
{
  "type": "message-start",
  "id": "cc5336e7-24f3-492d-a87c-d473907feb2c",
  "delta": {
    "message": {
      "role": "assistant",
      "content": [],
      "tool_plan": "",
      "tool_calls": [],
      "citations": []
    }
  }
}
```

#### Content Start
```json
{
  "type": "content-start",
  "index": 0,
  "delta": {
    "message": {
      "content": {
        "type": "text",
        "text": ""
      }
    }
  }
}
```

#### Content Delta (Token by Token)
```json
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":"Hello"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":"!"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" How"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" can"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" I"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" help"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" you"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":" today"}}}}
{"type":"content-delta","index":0,"delta":{"message":{"content":{"text":"?"}}}}
```

#### End of Content
```json
{
  "type": "content-end",
  "index": 0
}
```

#### End of Message
```json
{
  "type": "message-end",
  "delta": {
    "finish_reason": "COMPLETE",
    "usage": {
      "billed_units": {
        "input_tokens": 3,
        "output_tokens": 9
      },
      "tokens": {
        "input_tokens": 209,
        "output_tokens": 9
      }
    }
  }
}
```


---

