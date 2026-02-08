# Contributing to Demos Network Documentation

This guide explains how to edit and contribute to the documentation.

## Quick Start

```bash
# Install dependencies
npm install

# Start local preview
npx mintlify dev

# Check for broken links before committing
npx mintlify broken-links
```

The local preview runs at http://localhost:3000 with hot reload.

## File Structure

```
documentation-mintlify/
├── docs.json              # Navigation and site configuration
├── images/                # All images and assets
├── introduction/          # Getting started docs
├── sdk/                   # SDK documentation
├── backend/               # Backend documentation
├── frontend/              # Frontend documentation
├── cookbook/              # Tutorials and guides
└── faq.mdx               # FAQ page
```

## Page Format (MDX)

Pages use MDX format - Markdown with JSX components. Every page needs frontmatter:

```mdx
---
title: "Page Title"
description: "Brief description for SEO (optional)"
icon: "icon-name"  # Optional, uses Font Awesome icons
---

# Your Content

Regular markdown works here.
```

## Markdown Basics

```mdx
# Heading 1
## Heading 2
### Heading 3

**bold text**
*italic text*
`inline code`

[Link text](https://example.com)
[Internal link](/sdk/overview)

- Bullet list
- Another item

1. Numbered list
2. Second item

| Column 1 | Column 2 |
|----------|----------|
| Cell 1   | Cell 2   |
```

## Code Blocks

````mdx
```typescript
const demo = new Demos()
await demo.connect("https://demosnode.discus.sh")
```

```python
print("Hello from Python")
```

```bash
npm install @kynesyslabs/demosdk
```
````

## Mintlify Components

### Callouts

```mdx
<Note>General information the reader should know.</Note>

<Tip>Helpful suggestions and best practices.</Tip>

<Info>Additional context or background information.</Info>

<Warning>Important warnings about potential issues.</Warning>
```

### Cards

```mdx
<Card title="Getting Started" icon="rocket" href="/sdk/getting-started">
  Learn the basics of the SDK
</Card>

<CardGroup cols={2}>
  <Card title="WebSDK" icon="globe" href="/sdk/websdk/overview">
    Browser-based SDK
  </Card>
  <Card title="Cross Chain" icon="link" href="/sdk/cross-chain/overview">
    Multi-chain support
  </Card>
</CardGroup>
```

### Images

```mdx
<!-- Simple image -->
![Alt text](/images/example.png)

<!-- Image with caption -->
<Frame caption="Descriptive caption for the image">
  <img src="/images/example.png" alt="Alt text" />
</Frame>
```

### Tabs

```
<Tabs>
  <Tab title="npm">
    npm install @kynesyslabs/demosdk
  </Tab>
  <Tab title="yarn">
    yarn add @kynesyslabs/demosdk
  </Tab>
  <Tab title="pnpm">
    pnpm add @kynesyslabs/demosdk
  </Tab>
</Tabs>
```

### Accordions

```mdx
<Accordion title="Click to see more details">
  This content is hidden until the user clicks.
</Accordion>

<AccordionGroup>
  <Accordion title="First Section">
    Content for first section
  </Accordion>
  <Accordion title="Second Section">
    Content for second section
  </Accordion>
</AccordionGroup>
```

### Steps

```mdx
<Steps>
  <Step title="Install the SDK">
    Run `npm install @kynesyslabs/demosdk`
  </Step>
  <Step title="Initialize">
    Create a new Demos instance
  </Step>
  <Step title="Connect">
    Connect to an RPC node
  </Step>
</Steps>
```

## Adding a New Page

1. **Create the file**: Add a new `.mdx` file in the appropriate directory

2. **Add frontmatter**: Include title and description at the top

3. **Update navigation**: Add the page to `docs.json`:

```json
{
  "group": "Your Section",
  "pages": [
    "path/to/existing-page",
    "path/to/your-new-page"  // Add your page here
  ]
}
```

4. **Verify**: Run `npx mintlify broken-links` to check for issues

## Adding Images

1. Place images in the `/images/` directory
2. Reference them with absolute paths: `/images/your-image.png`
3. Use descriptive filenames: `auth-flow-diagram.png` not `img1.png`

## Navigation Structure (docs.json)

The `docs.json` file controls the entire site structure:

```json
{
  "navigation": {
    "tabs": [
      {
        "tab": "Tab Name",
        "groups": [
          {
            "group": "Section Name",
            "icon": "icon-name",
            "pages": [
              "folder/page-name",
              {
                "group": "Nested Section",
                "pages": [
                  "folder/nested/page1",
                  "folder/nested/page2"
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

## Icons

Icons use [Font Awesome](https://fontawesome.com/icons). Common icons:

- `rocket` - Getting started
- `code` - Code/SDK
- `book` - Documentation
- `gear` - Settings/Config
- `database` - Storage
- `link` - Connections
- `shield-check` - Security
- `flask` - Experimental

## Best Practices

1. **Keep titles concise** - Under 60 characters for SEO
2. **Add descriptions** - Helps with search and previews
3. **Use meaningful headings** - Structure content logically
4. **Include code examples** - Developers love copy-paste examples
5. **Test locally** - Always preview before pushing
6. **Check links** - Run `npx mintlify broken-links` regularly

## Common Issues

### MDX Parse Errors

Angle brackets in text are interpreted as JSX. Escape them:

```mdx
<!-- Bad -->
Response time <100ms

<!-- Good -->
Response time less than 100ms

<!-- Or use code -->
Response time `<100ms`
```

### Broken Internal Links

- Use absolute paths: `/sdk/overview` not `./overview`
- Don't include `.mdx` extension in links
- Check that the page exists in `docs.json`

### Images Not Loading

- Ensure image is in `/images/` directory
- Use absolute path: `/images/file.png`
- Check filename case sensitivity

## Deployment

Documentation is automatically deployed when changes are pushed to the main branch (if configured with Mintlify hosting).

For manual builds:
```bash
npx mintlify build
```

## Getting Help

- [Mintlify Documentation](https://mintlify.com/docs)
- [MDX Documentation](https://mdxjs.com/)
- Internal: Ask in #documentation channel
