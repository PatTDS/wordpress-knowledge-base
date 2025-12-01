---
title: Schema Markup and Structured Data Reference
description: Complete reference for implementing schema.org structured data in WordPress
category: seo
type: reference
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - schema
  - structured-data
  - rich-snippets
  - json-ld
  - seo
audience: intermediate
sources:
  - https://aioseo.com/how-to-implement-structured-data-in-wordpress/
  - https://yoast.com/structured-data-schema-ultimate-guide/
  - https://kinsta.com/blog/schema-markup-wordpress/
  - https://belovdigital.agency/blog/how-to-implement-schema-markup-in-wordpress/
  - https://wpengine.com/resources/schema-wordpress-optimize-seo/
  - https://schema.org/
---

# Schema Markup and Structured Data Reference

## Overview

Schema markup is a vocabulary of structured data that helps search engines understand your content. It enables rich results (rich snippets) in search results, improving visibility and click-through rates.

## Format Comparison

| Format | Recommendation | Notes |
|--------|---------------|-------|
| JSON-LD | Preferred by Google, Bing, Yoast | Easy to maintain, script-based |
| Microdata | Legacy support | Inline HTML attributes |
| RDFa | Legacy support | Less common |

**Always use JSON-LD** - it's the format recommended by Google and all major SEO plugins.

## Common Schema Types

### Article Schema

For blog posts and news articles.

```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "How to Implement Schema Markup",
  "description": "A complete guide to structured data",
  "image": "https://example.com/image.jpg",
  "author": {
    "@type": "Person",
    "name": "John Doe",
    "url": "https://example.com/author/john-doe/"
  },
  "publisher": {
    "@type": "Organization",
    "name": "Site Name",
    "logo": {
      "@type": "ImageObject",
      "url": "https://example.com/logo.png"
    }
  },
  "datePublished": "2025-12-01",
  "dateModified": "2025-12-01"
}
```

### LocalBusiness Schema

For local businesses with physical locations.

```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Business Name",
  "description": "Business description",
  "image": "https://example.com/image.jpg",
  "url": "https://example.com",
  "telephone": "+1-555-123-4567",
  "email": "contact@example.com",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "123 Main St",
    "addressLocality": "City",
    "addressRegion": "State",
    "postalCode": "12345",
    "addressCountry": "US"
  },
  "geo": {
    "@type": "GeoCoordinates",
    "latitude": 40.7128,
    "longitude": -74.0060
  },
  "openingHoursSpecification": [
    {
      "@type": "OpeningHoursSpecification",
      "dayOfWeek": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday"],
      "opens": "09:00",
      "closes": "17:00"
    }
  ],
  "priceRange": "$$"
}
```

### Product Schema

For e-commerce product pages.

```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "Product Name",
  "description": "Product description",
  "image": "https://example.com/product.jpg",
  "brand": {
    "@type": "Brand",
    "name": "Brand Name"
  },
  "sku": "SKU123",
  "mpn": "MPN456",
  "offers": {
    "@type": "Offer",
    "price": "99.99",
    "priceCurrency": "USD",
    "availability": "https://schema.org/InStock",
    "url": "https://example.com/product",
    "priceValidUntil": "2025-12-31"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "42"
  }
}
```

### FAQPage Schema

For FAQ sections and pages.

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What is schema markup?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Schema markup is structured data that helps search engines understand your content."
      }
    },
    {
      "@type": "Question",
      "name": "Why is schema important for SEO?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Schema can enable rich results, improving visibility and click-through rates."
      }
    }
  ]
}
```

### HowTo Schema

For step-by-step guides and tutorials.

```json
{
  "@context": "https://schema.org",
  "@type": "HowTo",
  "name": "How to Install WordPress",
  "description": "A guide to installing WordPress",
  "image": "https://example.com/howto.jpg",
  "totalTime": "PT30M",
  "estimatedCost": {
    "@type": "MonetaryAmount",
    "currency": "USD",
    "value": "0"
  },
  "step": [
    {
      "@type": "HowToStep",
      "name": "Download WordPress",
      "text": "Download the latest version from wordpress.org",
      "image": "https://example.com/step1.jpg"
    },
    {
      "@type": "HowToStep",
      "name": "Upload to Server",
      "text": "Upload the files to your web server",
      "image": "https://example.com/step2.jpg"
    }
  ]
}
```

### BreadcrumbList Schema

For navigation breadcrumbs.

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "https://example.com/"
    },
    {
      "@type": "ListItem",
      "position": 2,
      "name": "Blog",
      "item": "https://example.com/blog/"
    },
    {
      "@type": "ListItem",
      "position": 3,
      "name": "Schema Markup Guide",
      "item": "https://example.com/blog/schema-guide/"
    }
  ]
}
```

### Review Schema

For product/service reviews.

```json
{
  "@context": "https://schema.org",
  "@type": "Review",
  "itemReviewed": {
    "@type": "Product",
    "name": "Product Name"
  },
  "author": {
    "@type": "Person",
    "name": "Reviewer Name"
  },
  "reviewRating": {
    "@type": "Rating",
    "ratingValue": "4",
    "bestRating": "5"
  },
  "reviewBody": "The review text goes here."
}
```

### Service Schema

For service-based businesses.

```json
{
  "@context": "https://schema.org",
  "@type": "Service",
  "name": "Web Development Service",
  "description": "Custom WordPress development",
  "provider": {
    "@type": "Organization",
    "name": "Company Name"
  },
  "serviceType": "Web Development",
  "areaServed": {
    "@type": "City",
    "name": "City Name"
  },
  "offers": {
    "@type": "Offer",
    "price": "1000",
    "priceCurrency": "USD"
  }
}
```

## Schema Types Available by Plugin

| Schema Type | Rank Math | Yoast SEO | AIOSEO |
|------------|-----------|-----------|--------|
| Article | Yes | Yes | Yes |
| LocalBusiness | Yes | Premium | Yes |
| Product | Yes | Premium | Yes |
| FAQPage | Yes | Yes | Yes |
| HowTo | Yes | Yes | Yes |
| Recipe | Yes | Premium | Yes |
| Video | Yes | Premium | Yes |
| Course | Yes | No | Yes |
| Event | Yes | Premium | Yes |
| Job Posting | Yes | No | Yes |
| Book | Yes | No | No |
| Music | Yes | No | No |
| Person | Yes | Yes | Yes |
| Organization | Yes | Yes | Yes |

## WordPress Implementation Methods

### Method 1: SEO Plugin (Recommended)

Most SEO plugins handle schema automatically:

**Rank Math:**
- Settings > Titles & Meta > Select post type > Schema Type
- Edit any post > Schema tab

**Yoast SEO:**
- Settings > Content Types > Select type > Schema
- Edit post > Yoast sidebar > Schema tab

**AIOSEO:**
- Search Appearance > Content Types > Schema Markup
- Edit post > AIOSEO Settings > Schema

### Method 2: Dedicated Schema Plugin

**Schema & Structured Data for WP & AMP:**
```bash
wp plugin install schema-and-structured-data-for-wp --activate
```

Features:
- 35+ schema types
- Automated schema generation
- Custom schema builder
- Schema testing integration

### Method 3: Manual Implementation

Add JSON-LD to your theme:

```php
// In functions.php
function add_custom_schema() {
    if (is_single()) {
        global $post;

        $schema = array(
            "@context" => "https://schema.org",
            "@type" => "Article",
            "headline" => get_the_title($post->ID),
            "description" => get_the_excerpt($post->ID),
            "datePublished" => get_the_date('c', $post->ID),
            "dateModified" => get_the_modified_date('c', $post->ID),
            "author" => array(
                "@type" => "Person",
                "name" => get_the_author_meta('display_name', $post->post_author)
            )
        );

        echo '<script type="application/ld+json">' . wp_json_encode($schema) . '</script>';
    }
}
add_action('wp_head', 'add_custom_schema');
```

### Method 4: WordPress Blocks (Gutenberg)

Yoast SEO provides schema blocks:
- FAQ Block (generates FAQPage schema)
- How-to Block (generates HowTo schema)

## Testing and Validation

### Google Tools

| Tool | Purpose | URL |
|------|---------|-----|
| Rich Results Test | Test rich result eligibility | https://search.google.com/test/rich-results |
| Schema Markup Validator | Validate JSON-LD syntax | https://validator.schema.org/ |
| URL Inspection Tool | Check indexed schema | Google Search Console |

### Validation Checklist

- [ ] No syntax errors in JSON-LD
- [ ] Required properties present for schema type
- [ ] URLs are absolute (not relative)
- [ ] Images meet size requirements (min 1200x630 for articles)
- [ ] No mismatched data (schema matches visible content)
- [ ] Correct date formats (ISO 8601)
- [ ] Valid organization/person entities

## Rich Result Eligibility

### Supported Rich Results (2024)

| Result Type | Required Schema |
|------------|-----------------|
| Article rich result | Article, NewsArticle |
| Breadcrumb | BreadcrumbList |
| FAQ dropdown | FAQPage |
| How-to | HowTo |
| Product snippets | Product + Review/AggregateRating |
| Recipe cards | Recipe |
| Review stars | Review, AggregateRating |
| Video results | VideoObject |
| Event listings | Event |
| Job postings | JobPosting |
| Local business | LocalBusiness + location |

### Schema Required Properties

**Article (minimum):**
- headline
- image
- datePublished
- author

**LocalBusiness (minimum):**
- name
- address

**Product (minimum):**
- name
- image
- offers (with price, priceCurrency, availability)

**FAQPage (minimum):**
- mainEntity (array of Question/Answer)

## Common Schema Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Missing required field | Property not included | Add required properties |
| Invalid URL | Relative URL used | Use absolute URLs |
| Invalid image | Image too small | Min 1200px width for articles |
| Content mismatch | Schema doesn't match visible content | Ensure consistency |
| Duplicate schema | Multiple schema blocks for same type | Remove duplicates |
| Invalid date format | Wrong date format | Use ISO 8601 (YYYY-MM-DD) |

## Best Practices

### Do's

- Use one primary schema type per page
- Keep schema consistent with visible content
- Include all required properties
- Use absolute URLs for all links
- Test before publishing
- Monitor rich results in Search Console

### Don'ts

- Don't use schema for hidden content
- Don't mark up content users can't see
- Don't use misleading schema
- Don't duplicate schema unnecessarily
- Don't use incorrect schema types

## Schema for Specific Use Cases

### E-commerce Sites

```
Homepage: Organization + WebSite
Category: CollectionPage + BreadcrumbList
Product: Product + BreadcrumbList + Review
Cart: Leave plain or use WebPage
```

### Local Business Sites

```
Homepage: LocalBusiness + Organization
About: Organization + Person (team)
Services: Service + BreadcrumbList
Contact: ContactPage + LocalBusiness
```

### Blog/News Sites

```
Homepage: WebSite + Organization
Post: Article/NewsArticle + BreadcrumbList
Author: Person + ProfilePage
Category: CollectionPage + BreadcrumbList
```

## AI and Structured Data (2025)

Structured data is becoming increasingly important for AI-driven search:

- Google's AI Overviews use structured data
- Rich results appear in AI-generated summaries
- Schema helps AI understand content context
- Well-structured content is more likely to be cited

## Related Documents

- @seo/ref-technical-seo.md - Technical SEO reference
- @seo/howto-seo-plugin-configuration.md - Plugin setup
- @seo/howto-local-seo.md - Local SEO implementation
