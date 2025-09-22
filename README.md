# Gemini-2.5-Flash-Image-Preview-Nano-Banana-n8n-Guide
I faced the problem of every API from Gemini gave me the error of exceeding quotas, this is a way to get over that pain

## Quick Setup

### HTTP Request Node Configuration

**Method**: `POST`

**URL**: 
```
https://aiplatform.googleapis.com/v1/projects/YOUR_PROJECT_ID/locations/global/publishers/google/models/gemini-2.5-flash-image-preview:generateContent
```

**Authentication**: 
- Type: `Predefined Credential Type`
- Credential: `Google Service Account` (recommended)

**Send Body**: `ON`
**Body Content Type**: `JSON`

### Required JSON Body

```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "text": "Your image generation prompt here"
        }
      ]
    }
  ],
  "generationConfig": {
    "temperature": 0.7,
    "maxOutputTokens": 2048,
    "responseModalities": ["TEXT", "IMAGE"]
  }
}
```

### Response Processing Code

```javascript
// Extract image from Gemini response
const response = $input.first().json;
const parts = response.candidates[0].content.parts;

// Find the image data
let imageBase64 = null;
for (const part of parts) {
  if (part.inlineData && part.inlineData.data) {
    imageBase64 = part.inlineData.data;
    break;
  }
}

if (!imageBase64) {
  throw new Error("No image data found in response");
}

// Convert to binary
const binaryData = Buffer.from(imageBase64, 'base64');
const fileName = `generated_image_${Date.now()}.png`;

return [{
  json: {
    fileName: fileName,
    success: true
  },
  binary: {
    data: {
      data: binaryData,
      mimeType: 'image/png',
      fileName: fileName
    }
  }
}];
```

## Common Errors & Solutions

### Error 1: "Please use a valid role: user, model"
**Cause**: Missing `"role": "user"` in contents
**Solution**: Add role field to contents object
```json
{
  "contents": [
    {
      "role": "user",  // ← This was missing
      "parts": [...]
    }
  ]
}
```

### Error 2: "The request is not supported by this model"
**Cause**: Missing `responseModalities` parameter
**Solution**: Add responseModalities to generationConfig
```json
{
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"]  // ← Required for image generation
  }
}
```

### Error 3: "Model not found" / 404 Error
**Cause**: Wrong model name or unavailable in region
**Solutions**:
- Use global endpoint: `/locations/global/`
- Verify model name: `gemini-2.5-flash-image-preview`
- Alternative: Try `gemini-2.0-flash-preview-image-generation`

### Error 4: "Quota exceeded" / 429 Error
**Cause**: Free tier limits or wrong authentication
**Solutions**:
- Use Service Account instead of OAuth
- Enable billing on Google Cloud project
- Add rate limiting (Wait node between requests)
- Verify project has proper billing setup

### Error 5: "Bad request - please check your parameters"
**Cause**: Usually JSON format issues
**Solutions**:
- Verify JSON syntax is valid
- Check all required fields are present
- Ensure proper escaping in dynamic content

## Authentication Setup

### Option 1: Service Account (Recommended)
1. Go to Google Cloud Console → IAM & Admin → Service Accounts
2. Create new service account
3. Add roles: "Vertex AI User", "Service Account Token Creator"
4. Download JSON key file
5. In n8n: Use "Service Account" credential, upload JSON file

### Option 2: OAuth2
1. Create OAuth2 credentials in Google Cloud Console
2. Add scope: `https://www.googleapis.com/auth/cloud-platform`
3. In n8n: Use "Google OAuth2" credential

## Important Parameters

### Required Parameters:
- `"role": "user"` - Identifies message sender
- `"responseModalities": ["TEXT", "IMAGE"]` - Enables image generation
- `"parts"` array with `"text"` field - Contains the prompt

### Optional Parameters:
- `temperature` (0.0-1.0) - Controls creativity
- `maxOutputTokens` - Limits response length
- `candidateCount` - Number of response variations (usually 1)

## Prompt Best Practices

### Good Prompt Structure:
```
Create a [style] [type] featuring [key elements].

Technical specifications: [requirements]
Visual style: [description]
Color scheme: [colors]
Layout: [arrangement]
```

### Example Prompt:
```
Create a professional minimalist poster design featuring a modern product showcase.

Technical specifications: High-resolution, 9:16 aspect ratio, clean composition
Visual style: Modern minimalist with bold typography
Color scheme: Blue and white with accent colors
Layout: Centered product image with text hierarchy below
```

## Testing & Debugging

### Test with Simple Prompt:
```json
{
  "contents": [
    {
      "role": "user",
      "parts": [
        {
          "text": "Create a simple red circle on white background"
        }
      ]
    }
  ],
  "generationConfig": {
    "responseModalities": ["TEXT", "IMAGE"]
  }
}
```

### Debug Checklist:
1. ✅ URL has correct project ID
2. ✅ Authentication is working
3. ✅ JSON includes `"role": "user"`
4. ✅ JSON includes `"responseModalities": ["TEXT", "IMAGE"]`
5. ✅ Prompt is clear and descriptive
6. ✅ Response processing handles `inlineData.data`

## Response Format

Successful response structure:
```json
{
  "candidates": [
    {
      "content": {
        "parts": [
          {
            "text": "Generated description..."
          },
          {
            "inlineData": {
              "mimeType": "image/png",
              "data": "base64-encoded-image-data"
            }
          }
        ]
      }
    }
  ]
}
```

## Cost & Limits

- **Cost**: $0.039 per image (1290 tokens)
- **Size**: 1024x1024px default
- **Format**: PNG
- **Rate Limits**: Varies by tier (use global endpoint for better limits)

## Alternative Endpoints

If nano banana is unavailable:

**Imagen 3** (Vertex AI):
```
/publishers/google/models/imagen-3.0-generate-002:predict
```

**Google AI Studio** (with API key):
```
https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-image-preview:generateContent
```
