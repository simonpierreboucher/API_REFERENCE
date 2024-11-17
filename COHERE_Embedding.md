# Cohere Embedding

The **Embed API** generates embeddings for input texts or images. An embedding is a vector of floating-point numbers capturing semantic information about the text or image it represents. These embeddings are useful for tasks like semantic search, classification, and clustering.

---

## Endpoint

**POST** `https://api.cohere.com/v2/embed`

---

## Purpose

Generate embeddings for input texts or images to enable tasks like:

- Text classification
- Semantic search
- Clustering

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

| Field              | Type             | Description                                                                                     |
|--------------------|------------------|-------------------------------------------------------------------------------------------------|
| `model`            | String           | The identifier of the embedding model to use. Defaults to `embed-english-v2.0`.                |
| `input_type`       | Enum             | Specifies the type of input. Values: `search_document`, `search_query`, `classification`, `clustering`, `image`. |
| `embedding_types`  | List of Enums    | The types of embeddings to return. Default is `["float"]`. Possible values: `float`, `int8`, `uint8`, `binary`, `ubinary`. |

### Optional Fields

| Field              | Type             | Description                                                                                     |
|--------------------|------------------|-------------------------------------------------------------------------------------------------|
| `texts`            | List of Strings  | Texts to embed. Maximum: 96 texts per request. Reduce each text to under 512 tokens for best quality. |
| `images`           | List of Strings  | Image data URIs to embed. Maximum: 1 image per request. Format: `image/jpeg` or `image/png`, max size: 5 MB. |
| `truncate`         | Enum             | How to handle inputs exceeding the token limit. Options: `NONE`, `START`, `END`. Default: `END`. |

---

## Models and Dimensions

| Model                          | Embedding Dimensions | Description                              |
|--------------------------------|----------------------|------------------------------------------|
| `embed-english-v3.0`           | 1024                | Larger model for high performance.       |
| `embed-multilingual-v3.0`      | 1024                | Multilingual model.                      |
| `embed-english-light-v3.0`     | 384                 | Smaller, faster model.                   |
| `embed-multilingual-light-v3.0`| 384                 | Smaller, faster multilingual model.      |
| `embed-english-v2.0`           | 4096                | Older model with larger dimensions.      |
| `embed-english-light-v2.0`     | 1024                | Lightweight version of `v2.0`.           |
| `embed-multilingual-v2.0`      | 768                 | Multilingual version of `v2.0`.          |

---

## Response

The API returns a JSON object containing embeddings and related metadata.

### Response Fields

| Field              | Type             | Description                                                                                     |
|--------------------|------------------|-------------------------------------------------------------------------------------------------|
| `id`               | String           | Unique identifier for the request.                                                             |
| `embeddings`       | Object           | Embedding data grouped by type (e.g., `float`, `int8`). Each type contains embeddings for all input texts/images. |
| `texts`            | List of Strings  | Original input texts.                                                                          |
| `images`           | List of Objects  | Metadata for embedded images (e.g., width, height).                                            |
| `meta`             | Object           | Metadata about the request, including API version and billing information.                     |

### Embedding Object

| Type          | Description                                                                                     |
|---------------|-------------------------------------------------------------------------------------------------|
| `float`       | Array of float embeddings for each input text or image.                                         |
| `int8`        | Array of signed int8 embeddings. Values between `-128` and `127`. Only for `v3` models.         |
| `uint8`       | Array of unsigned int8 embeddings. Values between `0` and `255`. Only for `v3` models.          |
| `binary`      | Packed signed binary embeddings. Length is 1/8 of the float embeddings length.                  |
| `ubinary`     | Packed unsigned binary embeddings. Length is 1/8 of the float embeddings length.                |

---

## Example Request

### cURL Command

```bash
curl --request POST \
  --url https://api.cohere.com/v2/embed \
  --header 'accept: application/json' \
  --header 'content-type: application/json' \
  --header "Authorization: Bearer $CO_API_KEY" \
  --data '{
    "model": "embed-english-v3.0",
    "texts": ["hello", "goodbye"],
    "input_type": "classification",
    "embedding_types": ["float"]
  }'
```

---

## Example Response

```json
{
  "id": "da6e531f-54c6-4a73-bf92-f60566d8d753",
  "embeddings": {
    "float": [
      [
        0.016296387,
        -0.008354187,
        -0.04699707,
        0.019134521,
        ...
      ],
      [
        0.04663086,
        -0.023239136,
        0.008163452,
        -0.03945923,
        ...
      ]
    ]
  },
  "texts": [
    "hello",
    "goodbye"
  ],
  "meta": {
    "api_version": {
      "version": "2",
      "is_experimental": true
    },
    "billed_units": {
      "input_tokens": 2
    },
    "warnings": [
      "You are using an experimental version, for more information please refer to https://docs.cohere.com/versioning-reference"
    ]
  }
}
```

---

### Explanation of Response

- **`embeddings.float`**: A list of embeddings for each input text or image.
- **`meta.api_version`**: Version information for the API.
- **`meta.billed_units`**: Billing details, e.g., input tokens processed.

---

## Key Notes

- **Input Limits**: Maximum of 96 texts or 1 image per request.
- **Performance**: Use `light` models for faster processing; use full models for higher quality embeddings.
- **Truncation**: Use `truncate` to handle long inputs effectively.

