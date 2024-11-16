# Create Embeddings API Documentation

The **Create Embeddings API** generates an embedding vector representing the input text. Embeddings are numerical vector representations of text that can be used for various machine learning and search tasks.

---

## Endpoint
```
POST https://api.openai.com/v1/embeddings
```

---

## Request Body

### Parameters

| Name               | Type             | Required | Description                                                                                                                                  |
|--------------------|------------------|----------|----------------------------------------------------------------------------------------------------------------------------------------------|
| **input**          | string or array  | Required | Input text to embed, encoded as a string or array of tokens. The input must not exceed the model's token limit. Multiple inputs can be sent as an array. |
| **model**          | string           | Required | ID of the model to use. For example, `text-embedding-ada-002`. Refer to the [List Models API](#) for available models.                       |
| **encoding_format**| string           | Optional | Defaults to `float`. Format for returning embeddings. Can be `float` or `base64`.                                                           |
| **dimensions**     | integer          | Optional | Number of dimensions for the output embeddings. Supported in `text-embedding-3` and later models.                                           |
| **user**           | string           | Optional | A unique identifier representing the end-user. Useful for monitoring and abuse detection.                                                   |

---

## Returns

- **A list of embedding objects**.

Each embedding object contains the following:

| Field           | Type    | Description                                                                                       |
|------------------|---------|---------------------------------------------------------------------------------------------------|
| **object**      | string  | Always `embedding`.                                                                              |
| **embedding**   | array   | The embedding vector as a list of floats.                                                        |
| **index**       | integer | The index of the embedding in the input array (for batch requests).                              |

### Token Usage Statistics

| Field             | Type    | Description                                                                                   |
|-------------------|---------|-----------------------------------------------------------------------------------------------|
| **prompt_tokens** | integer | Number of tokens in the input text.                                                           |
| **total_tokens**  | integer | Total tokens used in the request.                                                             |

---

## Example cURL Request

This example generates an embedding for the input text *"The food was delicious and the waiter..."* using the `text-embedding-ada-002` model.

```bash
curl https://api.openai.com/v1/embeddings \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "input": "The food was delicious and the waiter...",
    "model": "text-embedding-ada-002",
    "encoding_format": "float"
  }'
```

---

## Example Response

```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [
        0.0023064255,
        -0.009327292,
        // ...(1536 floats total for ada-002)
        -0.0028842222
      ],
      "index": 0
    }
  ],
  "model": "text-embedding-ada-002",
  "usage": {
    "prompt_tokens": 8,
    "total_tokens": 8
  }
}
```

---

## Notes

1. **Embedding Dimensions**:
   - The output vector has 1536 dimensions for `text-embedding-ada-002`.
   - For models supporting configurable dimensions, use the `dimensions` parameter.

2. **Batch Requests**:
   - Multiple inputs can be provided as an array of strings. Each input will have its embedding object in the response.

3. **Encoding Format**:
   - Default format is `float` (list of floating-point numbers).
   - Use `base64` for compressed representations.

4. **Token Limits**:
   - Input tokens must not exceed the model's limit. For `text-embedding-ada-002`, this is 8192 tokens.

5. **User Identification**:
   - The `user` parameter can be helpful for tracking usage or handling abuse scenarios.
