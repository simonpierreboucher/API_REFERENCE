
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




# Create Image Edit API Documentation

The **Create Image Edit API** generates edited or extended images by modifying an original image based on a prompt. This API supports masking to specify the areas of the image to edit.

---

## Endpoint
```
POST https://api.openai.com/v1/images/edits
```

---

## Request Body

### Parameters

| Name               | Type            | Required | Description                                                                                                                                     |
|--------------------|-----------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| **image**          | file            | Required | The image to edit. Must be a valid PNG file, less than 4MB, and square. If no `mask` is provided, the image's transparency is used as the mask. |
| **prompt**         | string          | Required | A text description of the desired edits. The maximum length is 1000 characters.                                                                |
| **mask**           | file            | Optional | An additional PNG file (less than 4MB, same dimensions as `image`) specifying areas to edit. Transparent areas indicate editable regions.       |
| **model**          | string          | Optional | Defaults to `dall-e-2`. The model to use for image editing. Currently, only `dall-e-2` is supported.                                            |
| **n**              | integer or null | Optional | Defaults to `1`. The number of edited images to generate. Must be between 1 and 10.                                                            |
| **size**           | string or null  | Optional | Defaults to `1024x1024`. The size of the edited image. Supported sizes: `256x256`, `512x512`, `1024x1024`.                                      |
| **response_format**| string or null  | Optional | Defaults to `url`. Format of the generated images. Options are `url` or `b64_json`. URLs are valid for 60 minutes.                             |
| **user**           | string          | Optional | A unique identifier for the end-user. Useful for monitoring and abuse detection.                                                               |

---

## Returns

The API returns a list of **image objects** containing the edited images.

### Image Object Structure

| Field        | Type    | Description                                                                 |
|--------------|---------|-----------------------------------------------------------------------------|
| **url**      | string  | A URL to the edited image. Valid for 60 minutes.                           |

---

## Example cURL Request

This example edits the image `otter.png` using the mask `mask.png` to generate two images of a baby sea otter wearing a beret.

```bash
curl https://api.openai.com/v1/images/edits \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -F image="@otter.png" \
  -F mask="@mask.png" \
  -F prompt="A cute baby sea otter wearing a beret" \
  -F n=2 \
  -F size="1024x1024"
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

1. **Input Image**:
   - The image must be a valid PNG file and square-shaped.
   - If no `mask` is provided, the transparent parts of the `image` are treated as editable regions.

2. **Mask**:
   - Use a mask file to explicitly define which areas of the image should be modified.
   - Fully transparent areas in the mask indicate the editable regions.

3. **Model**:
   - Currently, only `dall-e-2` is supported for image editing.

4. **Image Size**:
   - Supported sizes: `256x256`, `512x512`, and `1024x1024`.

5. **Number of Outputs**:
   - You can request between 1 and 10 images using the `n` parameter.

6. **Response Format**:
   - Use `url` for temporary links or `b64_json` for base64-encoded image data.

7. **Usage**:
   - Ensure both the `image` and `mask` files are correctly formatted PNGs to avoid errors.

---
