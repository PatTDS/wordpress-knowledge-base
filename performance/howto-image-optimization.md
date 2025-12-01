---
title: How to Optimize Images for WordPress
description: Complete guide to image optimization including format selection (WebP, AVIF), compression, responsive images, and lazy loading
category: performance
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - images
  - webp
  - avif
  - lazy-loading
  - lcp
  - performance
prerequisites:
  - WordPress 6.5+ (for native AVIF support)
  - ImageMagick or GD library on server
audience: beginner
sources:
  - https://elementor.com/blog/webp-vs-avif/
  - https://instawp.com/optimize-images-for-wordpress/
  - https://luminosagency.com/webp-vs-avif-on-wordpress-image-speed-in-2025/
  - https://imagify.io/blog/image-lazy-loading-wordpress/
  - https://imgkonvert.com/blog/wordpress-image-optimization-guide
related:
  - concept-core-web-vitals.md
  - ref-performance-targets.md
---

# How to Optimize Images for WordPress

Images cause 80% of LCP problems on WordPress sites. On 73% of mobile pages, the LCP element is an image. Optimizing images is the highest-impact performance improvement you can make.

## Understanding Modern Image Formats

### Format Comparison

| Format | Compression vs JPEG | Browser Support | Best For |
|--------|---------------------|-----------------|----------|
| JPEG | Baseline | 100% | Photos (fallback) |
| WebP | 25-34% smaller | 97%+ | General use (2025) |
| AVIF | ~50% smaller | 92%+ | Maximum compression |
| PNG | Lossless | 100% | Graphics, transparency |

### WordPress Native Support

- **WebP:** Supported since WordPress 5.8 (July 2021)
- **AVIF:** Supported since WordPress 6.5 (April 2024)

**Requirements:** Server must have ImageMagick with WebP/AVIF support or GD library with WebP.

## Part 1: Automatic Optimization with Plugins

### Option A: ShortPixel (Recommended)

Free tier: 100 images/month (generous for maintenance)

```bash
wp plugin install shortpixel-image-optimiser --activate
```

**Configuration:**

```bash
# Via WP-CLI after getting API key
wp option update shortpixel_api_key "YOUR_API_KEY"
wp option update shortpixel_compression_type 2  # Lossy
wp option update shortpixel_auto_media_library 1
wp option update shortpixel_webp 1
wp option update shortpixel_retina 1
wp option update shortpixel_remove_exif 1
```

**Recommended Settings in wp-admin:**
1. Compression: Lossy (best compression, minimal quality loss)
2. Create WebP: Yes
3. Create AVIF: Yes (if server supports)
4. Remove EXIF: Yes (reduces file size)
5. Resize large images: Max 2560px width

### Option B: Imagify

```bash
wp plugin install imagify --activate
```

**Key Settings:**
- Optimization level: Aggressive
- Create WebP versions: Yes
- Display images in WebP: Yes
- Resize larger images: 2560px

### Option C: EWWW Image Optimizer

```bash
wp plugin install ewww-image-optimizer --activate
```

**Advantages:**
- Can run optimization locally (no API limits)
- Includes WebP delivery built-in

### Option D: Smush (by WPMU DEV)

```bash
wp plugin install wp-smushit --activate
```

**Features:**
- Free lazy loading
- Resize detection
- WebP conversion (Pro)

## Part 2: Manual Image Optimization

### Before Upload

Optimize images before uploading for best results:

```bash
# Install ImageMagick
sudo apt install imagemagick

# Optimize JPEG (quality 85, strip metadata)
convert input.jpg -strip -quality 85 -interlace Plane output.jpg

# Convert to WebP
cwebp -q 80 input.jpg -o output.webp

# Convert to AVIF
avifenc -c aom -s 6 -j all -q 80 input.jpg output.avif

# Batch convert to WebP
for file in *.jpg; do
  cwebp -q 80 "$file" -o "${file%.jpg}.webp"
done
```

### Recommended Pre-Upload Sizes

| Use Case | Max Width | Max File Size |
|----------|-----------|---------------|
| Hero image | 1920px | 150 KB |
| Blog featured | 1200px | 80 KB |
| In-content | 800px | 50 KB |
| Thumbnail | 300px | 20 KB |
| Logo | 400px | 15 KB |

## Part 3: Serving WebP/AVIF

### Method 1: Plugin-Based (Easiest)

Most optimization plugins handle this automatically with `<picture>` elements:

```html
<picture>
  <source srcset="image.avif" type="image/avif">
  <source srcset="image.webp" type="image/webp">
  <img src="image.jpg" alt="Description">
</picture>
```

### Method 2: .htaccess Rewrite (Apache)

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On

  # Check if browser accepts WebP
  RewriteCond %{HTTP_ACCEPT} image/webp

  # Check if WebP version exists
  RewriteCond %{DOCUMENT_ROOT}/$1.webp -f

  # Rewrite JPEG/PNG to WebP
  RewriteRule ^(.+)\.(jpe?g|png)$ $1.webp [T=image/webp,E=accept:1,L]
</IfModule>

<IfModule mod_headers.c>
  Header append Vary Accept env=REDIRECT_accept
</IfModule>

AddType image/webp .webp
```

### Method 3: Nginx Configuration

```nginx
location ~* ^.+\.(png|jpe?g)$ {
  add_header Vary Accept;

  if ($http_accept ~* "webp") {
    set $webp "A";
  }

  if (-f $request_filename.webp) {
    set $webp "${webp}B";
  }

  if ($webp = "AB") {
    rewrite ^(.*)$ $1.webp break;
  }
}
```

## Part 4: Lazy Loading

### Native WordPress Lazy Loading

WordPress 5.5+ adds `loading="lazy"` automatically to images. Verify in theme:

```php
// Check if native lazy loading is enabled (should be by default)
add_filter('wp_lazy_loading_enabled', '__return_true');

// WordPress adds this automatically:
<img src="image.jpg" loading="lazy" width="800" height="600">
```

### Exclude Above-Fold Images from Lazy Loading

**Critical:** Never lazy load LCP images (hero, featured images above fold).

```php
// Skip lazy loading for featured images
add_filter('wp_img_tag_add_loading_attr', function($value, $image, $context) {
    // Skip for featured images
    if (strpos($image, 'wp-post-image') !== false) {
        return false;
    }

    // Skip for images with fetchpriority="high"
    if (strpos($image, 'fetchpriority="high"') !== false) {
        return false;
    }

    return $value;
}, 10, 3);
```

### Prioritize LCP Images

```php
// Add fetchpriority to hero images
add_filter('wp_get_attachment_image_attributes', function($attr, $attachment, $size) {
    // For hero images
    if ($size === 'full' || $size === 'hero') {
        $attr['fetchpriority'] = 'high';
        unset($attr['loading']);  // Remove lazy loading
    }
    return $attr;
}, 10, 3);
```

In templates, explicitly set priority:

```html
<img src="hero.jpg"
     fetchpriority="high"
     width="1920"
     height="1080"
     alt="Hero image">
```

## Part 5: Responsive Images

### WordPress Automatic srcset

WordPress generates responsive images automatically:

```html
<img src="image-1024x768.jpg"
     srcset="image-300x225.jpg 300w,
             image-768x576.jpg 768w,
             image-1024x768.jpg 1024w,
             image-1536x1152.jpg 1536w"
     sizes="(max-width: 1024px) 100vw, 1024px"
     alt="Responsive image">
```

### Custom sizes Attribute

Default `sizes="(max-width: XXXpx) 100vw, XXXpx"` is often inefficient.

```php
// Customize sizes for your layout
add_filter('wp_calculate_image_sizes', function($sizes, $size, $image_src, $image_meta) {
    // For full-width images
    if ($size[0] >= 1200) {
        return '(max-width: 640px) 100vw, (max-width: 1024px) 50vw, 1200px';
    }

    // For sidebar images
    if ($size[0] <= 400) {
        return '(max-width: 640px) 100vw, 300px';
    }

    return $sizes;
}, 10, 4);
```

### Register Custom Image Sizes

```php
// In functions.php
add_action('after_setup_theme', function() {
    add_image_size('hero-mobile', 640, 360, true);
    add_image_size('hero-tablet', 1024, 576, true);
    add_image_size('hero-desktop', 1920, 1080, true);
    add_image_size('card', 400, 300, true);
});
```

## Part 6: Preventing CLS from Images

### Always Specify Dimensions

```html
<!-- Good: Dimensions prevent layout shift -->
<img src="image.jpg" width="800" height="600" alt="Description">

<!-- Bad: No dimensions causes CLS -->
<img src="image.jpg" alt="Description">
```

WordPress 5.5+ adds dimensions automatically. Verify your theme isn't stripping them.

### CSS Aspect Ratio (Modern Approach)

```css
.image-container {
  aspect-ratio: 16 / 9;
  width: 100%;
}

.image-container img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
```

### Use Placeholder Skeleton

```css
.lazy-image {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: skeleton 1.5s infinite;
}

@keyframes skeleton {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}
```

## Part 7: Bulk Optimization

### Existing Media Library

```bash
# ShortPixel bulk optimize
wp shortpixel optimize --limit=100

# EWWW optimize existing images
wp ewwwio optimize_local

# Regenerate thumbnails (after changing sizes)
wp media regenerate --yes
```

### Batch CLI Optimization

```bash
#!/bin/bash
# optimize-uploads.sh

UPLOADS_DIR="wp-content/uploads"

# Find unoptimized JPEGs (over 200KB)
find $UPLOADS_DIR -type f -name "*.jpg" -size +200k | while read file; do
  # Optimize JPEG
  convert "$file" -strip -quality 82 -interlace Plane "$file"

  # Create WebP version
  cwebp -q 80 "$file" -o "${file%.jpg}.webp"

  echo "Optimized: $file"
done
```

## Part 8: CDN Image Optimization

### Cloudflare Polish + WebP

Enable in Cloudflare dashboard:
- Speed → Optimization → Image Optimization → Polish
- Choose "Lossy" for best compression
- Enable "WebP" conversion

### Bunny Optimizer

```php
// Bunny CDN image optimization
// Images served through: https://yourzone.b-cdn.net/image.jpg?width=800&quality=80
```

### Optimole (Plugin + CDN)

```bash
wp plugin install optimole-wp --activate
```

Features:
- Automatic format conversion
- Responsive sizing
- Lazy loading
- Global CDN delivery

## Verification Checklist

- [ ] WebP versions being served (check Network tab)
- [ ] Images have width/height attributes
- [ ] Hero images NOT lazy loaded
- [ ] Hero images have `fetchpriority="high"`
- [ ] Largest image < 150KB
- [ ] Total page images < 1MB
- [ ] CLS score < 0.1

## Troubleshooting

### WebP Not Serving

1. Check browser support (Chrome DevTools → Network → Type column)
2. Verify .htaccess rules
3. Check plugin settings
4. Verify WebP files exist on server

### Images Still Large

1. Check if optimization plugin is active
2. Re-optimize existing images
3. Verify compression settings (use Lossy)
4. Check source image size before upload

### LCP Still Slow

1. Ensure hero image is preloaded:
   ```html
   <link rel="preload" as="image" href="hero.webp" type="image/webp">
   ```
2. Use CDN for image delivery
3. Reduce hero image dimensions
4. Consider inline Base64 for tiny critical images
