---
title: ImageMagick Image Processing Reference
category: tools
type: reference
updated: 2025-12-01
version: 1.0.0
tags:
  - imagemagick
  - image-optimization
  - webp
  - performance
sources:
  - https://github.com/ImageMagick/ImageMagick/discussions/3825
  - https://wordpress.org/plugins/imagemagick-engine/
  - https://www.smashingmagazine.com/2015/06/efficient-image-resizing-with-imagemagick/
  - https://wordpress.org/plugins/ewww-image-optimizer/
---

# ImageMagick Image Processing Reference

ImageMagick provides advanced image processing capabilities for WordPress. Version: **7.1.1+**

## Installation

### Ubuntu/Debian

```bash
sudo apt update
sudo apt install imagemagick
```

### macOS

```bash
brew install imagemagick
```

### Verify Installation

```bash
convert --version
# or
magick --version
```

## WordPress Integration

### ImageMagick vs GD Library

| Feature | ImageMagick | GD Library |
|---------|------------|------------|
| Quality | Superior | Basic |
| Formats | 200+ | ~10 |
| Metadata | Preserved | Destroyed |
| Memory | Higher | Lower |
| Color profiles | Preserved | Lost |

### Enable in WordPress

WordPress uses ImageMagick (via Imagick PHP extension) by default if available:

```bash
# Check if Imagick is installed
php -m | grep imagick

# Install Imagick for PHP
sudo apt install php-imagick
sudo systemctl restart apache2  # or php-fpm
```

### WordPress Plugins

**ImageMagick Engine** (updated October 2025):
- Prioritize quality or size per image size
- Preserve color profiles
- Regenerate existing images

**EWWW Image Optimizer** (February 2025):
- Uses ImageMagick for WebP conversion by default
- Automatic palette PNG correction

## Command Reference

### Basic Conversion

```bash
# Convert format
convert input.jpg output.png

# Convert with quality
convert input.jpg -quality 85 output.jpg

# Resize to width (maintain aspect ratio)
convert input.jpg -resize 800x output.jpg

# Resize to height
convert input.jpg -resize x600 output.jpg

# Resize to exact dimensions (may distort)
convert input.jpg -resize 800x600! output.jpg

# Resize only if larger
convert input.jpg -resize 800x600\> output.jpg
```

### WordPress-Optimized Settings

```bash
# Optimal JPEG compression (77% smaller files)
convert input.jpg \
  -filter Triangle \
  -define filter:support=2 \
  -thumbnail 1200x \
  -unsharp 0.25x0.25+8+0.065 \
  -dither None \
  -posterize 136 \
  -quality 82 \
  -define jpeg:fancy-upsampling=off \
  -interlace none \
  -colorspace sRGB \
  -strip \
  output.jpg
```

### WebP Conversion

```bash
# Convert to WebP
convert input.jpg -quality 80 output.webp

# Batch convert directory
for file in *.jpg; do
  convert "$file" -quality 80 "${file%.*}.webp"
done

# Using cwebp (Google's tool, often better)
cwebp -q 80 input.jpg -o output.webp
```

### PNG Optimization

```bash
# Optimize PNG
convert input.png \
  -strip \
  -define png:compression-filter=5 \
  -define png:compression-level=9 \
  -define png:compression-strategy=1 \
  output.png

# Convert to indexed colors (smaller file)
convert input.png -colors 256 output.png
```

### Batch Processing

```bash
# Resize all JPEGs in directory
mogrify -resize 1200x1200\> -quality 85 *.jpg

# Convert all to WebP
for file in *.{jpg,jpeg,png}; do
  convert "$file" -quality 80 "${file%.*}.webp"
done

# Process with progress
find . -name "*.jpg" | while read file; do
  echo "Processing: $file"
  convert "$file" -strip -quality 85 "$file"
done
```

### Metadata Handling

```bash
# Strip all metadata (smaller files)
convert input.jpg -strip output.jpg

# Preserve color profile only
convert input.jpg -strip +profile "icc" output.jpg

# View metadata
identify -verbose input.jpg
```

### Thumbnail Generation

```bash
# Create thumbnail (fast)
convert input.jpg -thumbnail 150x150^ \
  -gravity center \
  -extent 150x150 \
  output.jpg

# High-quality thumbnail
convert input.jpg \
  -auto-orient \
  -thumbnail 150x150^ \
  -gravity center \
  -extent 150x150 \
  -quality 85 \
  output.jpg
```

## WordPress Automation Scripts

### Optimize Uploads Directory

```bash
#!/bin/bash
# optimize-uploads.sh

UPLOADS_DIR="wp-content/uploads"
QUALITY=85

echo "Optimizing images in $UPLOADS_DIR..."

# Optimize JPEGs
find "$UPLOADS_DIR" -type f \( -name "*.jpg" -o -name "*.jpeg" \) | while read file; do
  echo "Optimizing: $file"
  convert "$file" \
    -strip \
    -interlace Plane \
    -quality $QUALITY \
    "$file"
done

# Optimize PNGs
find "$UPLOADS_DIR" -type f -name "*.png" | while read file; do
  echo "Optimizing: $file"
  convert "$file" \
    -strip \
    "$file"
done

echo "Done!"
```

### Generate WebP Versions

```bash
#!/bin/bash
# generate-webp.sh

UPLOADS_DIR="wp-content/uploads"
QUALITY=80

find "$UPLOADS_DIR" -type f \( -name "*.jpg" -o -name "*.jpeg" -o -name "*.png" \) | while read file; do
  webp_file="${file%.*}.webp"
  if [ ! -f "$webp_file" ]; then
    echo "Creating WebP: $webp_file"
    cwebp -q $QUALITY "$file" -o "$webp_file" 2>/dev/null
  fi
done
```

### Pre-Upload Optimization

```bash
#!/bin/bash
# pre-upload-optimize.sh

INPUT="$1"
OUTPUT="$2"
MAX_WIDTH=2048
QUALITY=85

if [ -z "$OUTPUT" ]; then
  OUTPUT="$INPUT"
fi

# Get image dimensions
width=$(identify -format "%w" "$INPUT")

if [ "$width" -gt "$MAX_WIDTH" ]; then
  # Resize and optimize
  convert "$INPUT" \
    -resize ${MAX_WIDTH}x \
    -strip \
    -quality $QUALITY \
    "$OUTPUT"
  echo "Resized and optimized: $OUTPUT"
else
  # Just optimize
  convert "$INPUT" \
    -strip \
    -quality $QUALITY \
    "$OUTPUT"
  echo "Optimized: $OUTPUT"
fi
```

## WordPress Upload Hook

```php
// Add to theme's functions.php or plugin

add_filter('wp_handle_upload', function($file) {
    if (strpos($file['type'], 'image') === 0) {
        $image_path = $file['file'];

        // Optimize based on type
        switch ($file['type']) {
            case 'image/jpeg':
            case 'image/jpg':
                // Optimize JPEG
                exec("convert '$image_path' -strip -quality 85 '$image_path'");
                break;

            case 'image/png':
                // Optimize PNG
                exec("convert '$image_path' -strip '$image_path'");
                break;
        }

        // Create WebP version
        $webp_path = preg_replace('/\.(jpg|jpeg|png)$/i', '.webp', $image_path);
        exec("cwebp -q 80 '$image_path' -o '$webp_path' 2>/dev/null");
    }

    return $file;
});
```

## Performance Targets

| Image Type | Target Size | Quality Setting |
|------------|-------------|-----------------|
| Hero images | < 200KB | 85 |
| Content images | < 100KB | 82 |
| Thumbnails | < 20KB | 80 |
| Icons/logos | < 10KB | 90 (PNG) |

## Common Issues

### PNG Size Increases

Problem: ImageMagick sometimes increases PNG file size.

Solution:
```bash
# Use pngquant for PNGs instead
pngquant --quality=65-80 --strip input.png

# Or optipng
optipng -o7 input.png
```

### Color Profile Issues

Problem: Colors look different after conversion.

Solution:
```bash
# Preserve sRGB color space
convert input.jpg -colorspace sRGB output.jpg

# Or convert to sRGB
convert input.jpg -profile sRGB.icc output.jpg
```

### Memory Errors

Problem: "cache resources exhausted"

Solution: Edit `/etc/ImageMagick-7/policy.xml`:
```xml
<policy domain="resource" name="memory" value="2GiB"/>
<policy domain="resource" name="map" value="4GiB"/>
<policy domain="resource" name="disk" value="10GiB"/>
```

## Best Practices

1. **Always backup** before batch processing
2. **Use correct quality** - 82-85 for JPEG, lossless for PNG
3. **Generate WebP** alongside original formats
4. **Strip metadata** for smaller files
5. **Resize before upload** - Max 2048px for web
6. **Test on staging** before production batch processing

## Related Documents

- @ref-wp-cli-commands.md
- @howto-docker-wordpress.md
- @howto-performance-optimization.md
