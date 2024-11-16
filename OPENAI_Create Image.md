
# Create Image API Documentation

The **Create Image API** generates images based on a textual prompt. It supports multiple models and configurations to produce customized visuals.

---

## Endpoint
```
POST https://api.openai.com/v1/images/generations
```

---

## Request Body

### Parameters

| Name               | Type            | Required | Description                                                                                                                                             |
|--------------------|-----------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| **prompt**         | string          | Required | A text description of the desired image(s). The maximum length is 1000 characters for `dall-e-2` and 4000 characters for `dall-e-3`.                    |
| **model**          | string          | Optional | Defaults to `dall-e-2`. The model to use for image generation.                                                                                          |
| **n**              | integer or null | Optional | Defaults to `1`. Number of images to generate (1-10). For `dall-e-3`, only `n=1` is supported.                                                          |
| **quality**        | string          | Optional | Defaults to `standard`. Determines the quality of the image. For `dall-e-3`, use `hd` for finer details and greater consistency.                        |
| **response_format**| string or null  | Optional | Defaults to `url`. Format of the generated images. Options are `url` or `b64_json`. URLs are valid for 60 minutes after the image is generated.          |
| **size**           | string or null  | Optional | Defaults to `1024x1024`. Specifies the size of the images. For `dall-e-2`, supported sizes: `256x256`, `512x512`, `1024x1024`. For `dall-e-3`, use `1024x1024`, `1792x1024`, or `1024x1792`. |
| **style**          | string or null  | Optional | Defaults to `vivid`. Determines the style of the generated image (`vivid` for hyper-realistic or `natural` for more subdued, realistic images). Only supported in `dall-e-3`. |
| **user**           | string          | Optional | A unique identifier for the end-user. Useful for tracking and abuse detection.                                                                          |

---

## Returns

The API returns a list of **image objects** containing the generated images.

### Image Object Structure

| Field        | Type    | Description                                                                 |
|--------------|---------|-----------------------------------------------------------------------------|
| **url**      | string  | A URL to the generated image. Valid for 60 minutes.                        |

---

## Example cURL Request

This example generates a single image of a baby sea otter using the `dall-e-3` model.

```bash
curl https://api.openai.com/v1/images/generations \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "dall-e-3",
    "prompt": "A cute baby sea otter",
    "n": 1,
    "size": "1024x1024"
  }'
```

---

## Example Response

```json
{
  "created": 1589478378,
  "data": [
    {
      "url": "https://..."
    },
    {
      "url": "https://..."
    }
  ]
}
```

---

## Notes

1. **Image Prompt**: The quality of the generated image depends on the clarity and descriptiveness of the prompt.
2. **Model Selection**:
   - Use `dall-e-3` for more advanced and realistic image generation.
   - For backward compatibility or smaller images, use `dall-e-2`.
3. **Image Sizes**:
   - Higher resolutions (e.g., `1024x1024`) result in more detailed outputs.
   - For wide or tall images, use sizes like `1792x1024` or `1024x1792` with `dall-e-3`.
4. **Quality and Style**:
   - Use `quality: hd` for higher-definition images with `dall-e-3`.
   - Experiment with `style: vivid` or `style: natural` for varying artistic effects.
5. **Response Format**:
   - Use `url` for temporary links or `b64_json` for base64-encoded image data.

