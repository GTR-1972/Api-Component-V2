# Discord Components v2 Complete Guide

## Overview

Discord Components v2 is a new system that provides more flexible layout and styling options compared to traditional embeds and message components. It allows you to create rich, interactive messages with various display components.

## Key Requirements & Limitations

### ✅ Requirements
- **Must set `IsComponentsV2` flag** when sending messages
- **Must use display components** for text content
- **All files must be explicitly referenced** in components

### ❌ Limitations
- **Cannot use** `content`, `embeds`, `stickers`, or `poll` fields
- **Maximum 40 components** per message (including nested)
- **Maximum 4000 characters** across all text display components
- **Cannot opt out** when editing a message that uses Components v2

## Using with discord.js

### Basic Setup

```javascript
const { TextDisplayBuilder, MessageFlags } = require('discord.js');

// Simple text display
const textDisplay = new TextDisplayBuilder()
    .setContent('Hello **world**! This supports markdown.');

await channel.send({
    components: [textDisplay],
    flags: MessageFlags.IsComponentsV2,
});
```

### Available Builders

| Builder | Purpose |
|---------|---------|
| `TextDisplayBuilder` | Basic text with markdown |
| `SectionBuilder` | Text with button/thumbnail accessory |
| `ContainerBuilder` | Grouped components in rounded box |
| `MediaGalleryBuilder` | Grid of up to 10 media items |
| `FileBuilder` | Display uploaded files |
| `SeparatorBuilder` | Visual spacing between components |
| `ThumbnailBuilder` | Image accessory for sections |

### Section with Button

```javascript
const { SectionBuilder, ButtonStyle, MessageFlags } = require('discord.js');

const section = new SectionBuilder()
    .addTextDisplayComponents(
        textDisplay => textDisplay.setContent('Main text'),
        textDisplay => textDisplay.setContent('Secondary text'),
    )
    .setButtonAccessory(
        button => button
            .setCustomId('my_button')
            .setLabel('Click Me')
            .setStyle(ButtonStyle.Primary),
    );

await channel.send({
    components: [section],
    flags: MessageFlags.IsComponentsV2,
});
```

### Container with Multiple Components

```javascript
const { ContainerBuilder, UserSelectMenuBuilder, MessageFlags } = require('discord.js');

const container = new ContainerBuilder()
    .setAccentColor(0x0099FF)
    .addTextDisplayComponents(
        textDisplay => textDisplay.setContent('Container title'),
    )
    .addActionRowComponents(
        actionRow => actionRow.setComponents(
            new UserSelectMenuBuilder()
                .setCustomId('user_select')
                .setPlaceholder('Select users'),
        ),
    )
    .addSeparatorComponents(separator => separator)
    .addSectionComponents(
        section => section
            .addTextDisplayComponents(
                textDisplay => textDisplay.setContent('Section in container'),
            )
            .setButtonAccessory(
                button => button
                    .setCustomId('section_button')
                    .setLabel('Section Button')
                    .setStyle(ButtonStyle.Secondary),
            ),
    );

await channel.send({
    components: [container],
    flags: MessageFlags.IsComponentsV2,
});
```

### Media Gallery

```javascript
const { AttachmentBuilder, MediaGalleryBuilder, MessageFlags } = require('discord.js');

const file = new AttachmentBuilder('../assets/image.png');

const gallery = new MediaGalleryBuilder()
    .addItems(
        item => item
            .setDescription('Local image')
            .setURL('attachment://image.png'),
        item => item
            .setDescription('External image')
            .setURL('https://example.com/image.jpg')
            .setSpoiler(true),
    );

await channel.send({
    components: [gallery],
    files: [file],
    flags: MessageFlags.IsComponentsV2,
});
```

## Using Direct Discord API

### Message Flag Value
The `IsComponentsV2` flag has a value of **1024** (or `0x400` in hexadecimal).

### Component Type Reference

| Type | Component | Description |
|------|-----------|-------------|
| 15 | Text Display | Basic text with markdown |
| 16 | Section | Text with button/thumbnail accessory |
| 17 | Thumbnail | Image accessory for sections |
| 18 | Media Gallery | Grid of media items |
| 19 | File | Display uploaded files |
| 20 | Separator | Visual spacing |
| 21 | Separator (alt) | Alternative separator |
| 22 | Container | Grouped components in box |

### Basic Text Display

```json
POST /api/v10/channels/{channel_id}/messages
{
  "flags": 1024,
  "components": [
    {
      "type": 15,
      "content": "This is a text display with **markdown** support!"
    }
  ]
}
```

### Section with Button

```json
{
  "flags": 1024,
  "components": [
    {
      "type": 16,
      "text_display_components": [
        {
          "type": 15,
          "content": "First text in section"
        },
        {
          "type": 15,
          "content": "Second text in section"
        }
      ],
      "button_accessory": {
        "type": 2,
        "custom_id": "example_button",
        "label": "Click me!",
        "style": 1
      }
    }
  ]
}
```

### Container Component

```json
{
  "flags": 1024,
  "components": [
    {
      "type": 22,
      "accent_color": 65535,
      "components": [
        {
          "type": 15,
          "content": "Text inside container"
        },
        {
          "type": 1,
          "components": [
            {
              "type": 5,
              "custom_id": "user_select",
              "placeholder": "Select users",
              "min_values": 1,
              "max_values": 1
            }
          ]
        },
        {
          "type": 21,
          "spacing": 1,
          "divider": true
        }
      ]
    }
  ]
}
```

### Media Gallery

```json
{
  "flags": 1024,
  "components": [
    {
      "type": 18,
      "items": [
        {
          "url": "attachment://image.png",
          "description": "Alt text for image",
          "spoiler": false
        },
        {
          "url": "https://example.com/external.jpg",
          "description": "External image",
          "spoiler": true
        }
      ]
    }
  ],
  "attachments": [
    {
      "id": "0",
      "filename": "image.png",
      "description": "An image file"
    }
  ]
}
```

### File Component

```json
{
  "flags": 1024,
  "components": [
    {
      "type": 19,
      "url": "attachment://document.pdf",
      "spoiler": false
    }
  ],
  "attachments": [
    {
      "id": "0",
      "filename": "document.pdf",
      "description": "A PDF document"
    }
  ]
}
```

### Separator Component

```json
{
  "flags": 1024,
  "components": [
    {
      "type": 15,
      "content": "First section"
    },
    {
      "type": 21,
      "spacing": 1,
      "divider": true
    },
    {
      "type": 15,
      "content": "Second section"
    }
  ]
}
```

## cURL Examples

### Basic Example

```bash
curl -X POST \
  -H "Authorization: Bot YOUR_BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "flags": 1024,
    "components": [
      {
        "type": 15,
        "content": "Hello from Components v2!"
      }
    ]
  }' \
  "https://discord.com/api/v10/channels/CHANNEL_ID/messages"
```

### With File Upload

```bash
curl -X POST \
  -H "Authorization: Bot YOUR_BOT_TOKEN" \
  -F "payload_json={\"flags\":1024,\"components\":[{\"type\":19,\"url\":\"attachment://file.pdf\"}],\"attachments\":[{\"id\":\"0\",\"filename\":\"file.pdf\"}]}" \
  -F "files[0]=@/path/to/file.pdf" \
  "https://discord.com/api/v10/channels/CHANNEL_ID/messages"
```

## Button Styles Reference

| Style | Value | Color |
|-------|-------|--------|
| Primary | 1 | Blue |
| Secondary | 2 | Grey |
| Success | 3 | Green |
| Danger | 4 | Red |
| Link | 5 | Grey (with URL) |

## Best Practices

1. **Always include the flag**: Set `flags: 1024` (discord.js) or `"flags": 1024` (API)
2. **Plan your layout**: Maximum 40 components including nested ones
3. **Keep text concise**: 4000 character limit across all text displays
4. **Use separators**: Add visual spacing between sections
5. **Reference files explicitly**: All attachments must be referenced in components
6. **Handle interactions**: Set up proper event handlers for buttons and selects
7. **Test thoroughly**: Components v2 has different behavior than traditional embeds

## Common Errors

- **Missing flag**: Message appears empty or fails to send
- **Using forbidden fields**: Cannot use `content`, `embeds`, `stickers`, or `poll`
- **Too many components**: Exceeds 40 component limit
- **Text too long**: Exceeds 4000 character limit
- **Unreferenced files**: Files not explicitly referenced in components

## Migration from Embeds

| Embed Feature | Components v2 Equivalent |
|---------------|-------------------------|
| `description` | `TextDisplayBuilder` |
| `color` | `ContainerBuilder.setAccentColor()` |
| `thumbnail` | `ThumbnailBuilder` in `SectionBuilder` |
| `image` | `MediaGalleryBuilder` |
| `fields` | Multiple `TextDisplayBuilder` components |
| `footer` | `TextDisplayBuilder` at the end |

Components v2 provides more flexibility but requires restructuring your message layout compared to traditional embeds.
