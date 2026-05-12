# Additional Channel Settings

This document describes extra channel configuration fields. These settings are usually provided as a JSON object.

## `force_format`

- Purpose: Forces the upstream response to be converted into the OpenAI-compatible format.
- Type: `boolean`
- Recommended value: Set to `true` only when the upstream provider does not return an OpenAI-compatible response by default.

## Maintenance Notes

Keep this configuration small and explicit. Avoid placing secrets, tokens, or internal-only URLs in public documentation.
