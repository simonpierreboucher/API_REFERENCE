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
