---
title: "Additional Channel Settings"
description: "Explains extra channel parameters such as force_format, proxy, and thinking_to_content."
---

# Additional Channel Settings

This configuration is used to set extra channel parameters. It can be provided as a JSON object and currently includes the following fields.

1. `force_format`
   - Indicates whether response data should be forcibly converted to the OpenAI-compatible format.
   - Type: `boolean`.
   - Set it to `true` to enable forced formatting.

2. `proxy`
   - Configures the network proxy used by the channel.
   - Type: `string`.
   - Use a proxy address such as a SOCKS5 proxy URL.

3. `thinking_to_content`
   - Indicates whether `reasoning_content` should be converted into a `<think>` tag and appended to the returned content.
   - Type: `boolean`.
   - Set it to `true` to enable reasoning-content conversion.

## JSON Example

The following example enables forced formatting, enables reasoning-content conversion, and sets a proxy address:

```json
{
  "force_format": true,
  "thinking_to_content": true,
  "proxy": "socks5://xxxxxxx"
}
```

By changing the values in this JSON configuration, you can control extra channel behavior, such as whether responses are formatted and whether a specific network proxy is used.
