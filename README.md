# Astro OG Image Generator Example

This project shows how to implement automatic Open Graph (OG) image generation
in Astro projects using Satori and Sharp. While this example focuses on a blog
use case, the implementation can be adapted for any type of website requiring
dynamic OG image generation.

**Features:**

- Automatic OG image generation at build time
- Customizable templates for posts and site-wide images
- Support for custom fonts
- Multiple image format options (JPEG, WebP, AVIF)
- Configurable image dimensions and quality settings

## Table of Contents

- [Project Structure](#-project-structure)
- [Configuration](#️-configuration)
  - [Template Customization](#template-customization)
  - [Creating Custom Templates](#creating-custom-templates)
  - [Image Configuration](#image-configuration)
- [Configuration Options Reference](#-configuration-options-reference)
  - [Font Configuration](#font-configuration)
  - [Image Format Options](#image-format-options)
  - [SVG Configuration](#svg-configuration)
- [How It Builds](#how-it-builds)
- [Usage](#-usage)

## 🚀 Project Structure

The project's OG image generation system is organized as follows:

```
src/
├── pages/
│   ├── blog/
│   │   └── [id].webp.ts        # Dynamic OG image generation for blog posts
│   └── og-image.webp.ts        # Main site OG image generation
└── utils/
    └── og-images/
        ├── templates/
        │   ├── postTemplate.ts   # Template for blog post OG images
        │   └── siteTemplate.ts   # Template for site-wide OG images
        ├── config.ts             # Configuration settings
        ├── generateOgImages.ts   # Main image generation logic
        └── types.ts              # TypeScript type definitions
```

## ⚙️ Configuration

### Template Customization

OG image templates are located in `src/utils/og-images/templates/`. You can
customize the appearance of your OG images by modifying these templates:

- `postTemplate.ts` - For individual post images
- `siteTemplate.ts` - For site-wide default images

Example of template modification:

```typescript
const markup = html(`
  // Your custom HTML template here
`);
```

On how to make these templates you can read here:

- [Satori](https://github.com/vercel/satori)
- [Satori HTML](https://github.com/natemoo-re/satori-html?tab=readme-ov-file)

### Creating Custom Templates

You can add your own templates by creating a new file in the templates folder
and updating `og-images/generateOgImages.ts` by importing the template and
adding new function following example below:

```typescript name=og-images/generateOgImages.ts
export async function yourFunctionName() {
  const svg = await yourTempalteName(yourParameters);

  return await svgBufferToImageBuffer(svg); // Convert SVG to image using Sharp
}
```

### Image Configuration

The image generation settings can be customized in
`src/utils/og-images/config.ts`:

```typescript name=src/utils/og-images/config.ts
import { readFileSync } from 'fs';
import type { ImageConfig, SvgConfig } from './types';
import type { Font } from 'satori';

// Import your fonts here
const interRegular = readFileSync(
  `${process.cwd()}/public/og-fonts/Inter-Regular.ttf`
);
const interBlack = readFileSync(
  `${process.cwd()}/public/og-fonts/Inter-Black.ttf`
);

// Image format configuration
export const DEFAULT_IMAGE: ImageConfig = {
  format: 'webp',
  webpOptions: {
    quality: 90,
  },
};

// SVG dimensions
export const DEFAULT_SVG: SvgConfig = {
  width: 1200,
  height: 630,
  embedFont: true,
};

// Font configuration
export const DEFAULT_FONTS: Font[] = [
  {
    name: 'Inter',
    data: interRegular,
    weight: 400,
    style: 'normal',
  },
  {
    name: 'Inter',
    data: interBlack,
    weight: 900,
    style: 'normal',
  },
];
```

## 📝 Configuration Options Reference

### Font Configuration

| Option   | Description                         |
| -------- | ----------------------------------- |
| `name`   | Font family name                    |
| `data`   | Font file data                      |
| `weight` | Font weight (400=regular, 900=bold) |
| `style`  | Font style ('normal'/'italic')      |

In this case we import fonts to `public/og-fonts`. While you might consider
loading them remotely from Google Fonts, following
[this issue](https://github.com/vercel/satori/issues/590), you'll likely end up
implementing them locally as demonstrated here.

### Image Format Options

| Format | Available Options                                          |
| ------ | ---------------------------------------------------------- |
| JPEG   | quality (1-100), chromaSubsampling, progressive, mozjpeg   |
| WebP   | quality (1-100), lossless, smartSubsample, effort (0-6)    |
| AVIF   | quality (1-100), lossless, chromaSubsampling, effort (0-9) |

These options are directly taken from [Sharp](https://sharp.pixelplumbing.com/).
You can add other formats by extending `og-images/types.ts`:

```typescript
import type { JpegOptions, WebpOptions, AvifOptions } from 'sharp';

// Add new formats here:
export type ImageFormat = 'jpeg' | 'webp' | 'avif';

export interface ImageConfig {
  format: ImageFormat;
  jpegOptions?: JpegOptions;
  webpOptions?: WebpOptions;
  avifOptions?: AvifOptions;
}
```

Then update the format in `og-images/config.ts`:

```typescript
export const DEFAULT_IMAGE: ImageConfig = {
  format: 'webp', // Change to your preferred format
  webpOptions: {
    quality: 90,
  },
};
```

### SVG Configuration

| Option      | Description                   |
| ----------- | ----------------------------- |
| `width`     | Image width (default: 1200px) |
| `height`    | Image height (default: 630px) |
| `embedFont` | Whether to embed fonts        |

## How It Builds

At build time, Astro will process everything in the pages folder (check (Astro
documentation)[https://docs.astro.build/en/guides/endpoints/] for more
information). We have two key files:

1. `og-image.webp.ts` - This builds the main site's OG image

   - During build, the output file `og-image.webp` will be in your dist folder
   - If you want a different format, rename to `og-image.avif.ts` or similar

2. `[id].webp.ts` in `pages/blog` - This generates OG images for blog posts
   - The `[id]` parameter will be replaced with each post's ID
   - Generates a unique webp image for each blog post in `dist/blog/[id].webp`

Here's an example of how the code looks inside these files:

```typescript name=og-image.webp.ts
import { generateOgImageForSite } from '@utils/og-images/generateOgImages';
import type { APIRoute } from 'astro';

export const GET: APIRoute = async () => {
  try {
    const image = await generateOgImageForSite();
    return new Response(image, {
      headers: {
        'Content-Type': 'image/webp',
      },
    });
  } catch (error) {
    console.error('Error generating OG image:', error);
    return new Response('Error generating OG image', { status: 500 });
  }
};
```

## 📚 Usage

1. Clone this repository
2. Install dependencies: `pnpm install`
3. Customize the templates in `src/utils/og-images/templates/`
4. Adjust the configuration in `src/utils/og-images/config.ts`
5. Run `pnpm build` to generate OG images automatically
