# VOYAGE Multimodal Embedding

The **Voyage Multimodal Embeddings API** generates vector representations for inputs containing text, images, or a combination of both. These embeddings can be used for retrieval, search, or clustering in applications requiring multimodal data handling.

---

## Endpoint
```
POST https://api.voyageai.com/v1/multimodalembeddings
```

---

## Authorization

**HTTP Authorization Scheme**: `bearer`  
**Header**:  
```
Authorization: Bearer <VOYAGE_API_KEY>
```

---

## Request Body Parameters

The request body must be sent in **JSON format** with the following fields:

### Required Parameters

| Parameter           | Type                | Description                                                                                                                                 |
|---------------------|---------------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| **inputs**          | array of objects    | A list of multimodal inputs, each containing a sequence of text and/or images. Each item must have a `content` key, explained below.        |
| **model**           | string              | The model name. Currently, only `"voyage-multimodal-3"` is supported.                                                                      |

### Optional Parameters

| Parameter           | Type                | Default        | Description                                                                                                                                 |
|---------------------|---------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------|
| **input_type**      | string              | null           | Input type for retrieval tasks. Options: `query`, `document`. See the **Notes** section for more details.                                  |
| **truncation**      | boolean             | true           | Whether to truncate inputs that exceed the context length. If `false`, an error is raised for over-length inputs.                          |
| **output_encoding** | string              | null           | Format for output embeddings. Options: `null` (list of floats), `base64` (Base64-encoded NumPy array).                                     |

---

## Input Format

The **inputs** parameter accepts a list of dictionaries. Each dictionary must have a `content` key with a list of dictionaries, representing the sequence of text and images.

### Content Dictionary Keys

| Key            | Type              | Description                                                                                 |
|----------------|-------------------|---------------------------------------------------------------------------------------------|
| **type**       | string            | The type of content. Allowed values: `text`, `image_url`, `image_base64`.                   |
| **text**       | string (optional) | The text string (if `type` is `text`).                                                     |
| **image_url**  | string (optional) | URL of the image (if `type` is `image_url`). Supported formats: PNG, JPEG, WEBP, GIF.       |
| **image_base64**| string (optional) | Base64-encoded image in `data:[<mediatype>];base64,<data>` format (if `type` is `image_base64`). |

### Notes on Image Usage
- Use **either** `image_url` or `image_base64` for images within a single request, not both.
- Image constraints:
  - Maximum 16 million pixels per image.
  - Maximum image size: 20 MB.

---

## Constraints

1. **List Size**:  
   - Maximum 1000 inputs per request.

2. **Token and Pixel Limits**:  
   - Each input:
     - Must not exceed **32,000 tokens**.
   - Total inputs:
     - Must not exceed **320,000 tokens**.  
     - Images: Every 560 pixels count as one token.

3. **Truncation**:
   - If `truncation: true`, over-length inputs are truncated to fit the context length. Entire images are discarded if truncation occurs mid-image.

---

## Example Request

### Example with Text and Image URL

```bash
curl -X POST https://api.voyageai.com/v1/multimodalembeddings \
  -H "Authorization: Bearer $VOYAGE_API_KEY" \
  -H "content-type: application/json" \
  -d '{
    "inputs": [
      {
        "content": [
          {
            "type": "text",
            "text": "This is a banana."
          },
          {
            "type": "image_url",
            "image_url": "https://raw.githubusercontent.com/voyage-ai/voyage-multimodal-3/refs/heads/main/images/banana.jpg"
          }
        ]
      }
    ],
    "model": "voyage-multimodal-3"
  }'
```

### Example with Base64 Image

```bash
curl -X POST https://api.voyageai.com/v1/multimodalembeddings \
  -H "Authorization: Bearer $VOYAGE_API_KEY" \
  -H "content-type: application/json" \
  -d '{
    "inputs": [
      {
        "content": [
          {
            "type": "text",
            "text": "This is a banana."
          },
          {
            "type": "image_base64",
            "image_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
          }
        ]
      }
    ],
    "model": "voyage-multimodal-3"
  }'
```

---

## Example Response

### Success Response (200)

| Field              | Type                | Description                                                                 |
|--------------------|---------------------|-----------------------------------------------------------------------------|
| **object**         | string              | Always `"list"`.                                                            |
| **data**           | array of objects    | List of embedding objects for each input.                                   |
| **model**          | string              | The model used for generating embeddings.                                   |
| **usage**          | object              | Token and pixel usage details: `text_tokens`, `image_pixels`, `total_tokens`. |

#### Response Example
```json
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "embedding": [
        0.027587891,
        -0.021240234,
        0.018310547,
        "...",
        -0.021240234
      ],
      "index": 0
    }
  ],
  "model": "voyage-multimodal-3",
  "usage": {
    "text_tokens": 5,
    "image_pixels": 2000000,
    "total_tokens": 3576
  }
}
```

---

## Notes

1. **Model Options**:
   - Currently, only `voyage-multimodal-3` is supported.

2. **Input Type (`input_type`)**:
   - **`query`**: Use for retrieval/search queries.
   - **`document`**: Use for relevant supporting information.
   - Prompts are prepended to the input for these modes to improve performance.

3. **Truncation**:
   - Ensure `truncation: true` for seamless processing of over-length inputs.
   - Images truncated mid-way are entirely discarded.

4. **Token Calculation**:
   - Every 560 image pixels count as **1 token**.

5. **Embedding Format**:
   - Default: List of floating-point numbers.
   - Use `output_encoding: "base64"` for compressed NumPy arrays.

6. **Error Handling**:
   - **4XX Errors**: Issues with request formatting or exceeding constraints.
   - **5XX Errors**: Server-side issues such as high traffic or unexpected failures.

---
