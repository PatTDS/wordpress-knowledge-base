---
title: How to Optimize JavaScript in WordPress
description: Complete guide to JavaScript optimization including defer, async, delay execution, reducing main thread work, and eliminating render-blocking scripts
category: performance
type: howto
created: 2025-12-01
updated: 2025-12-01
version: 1.0.0
status: stable
tags:
  - javascript
  - defer
  - async
  - inp
  - main-thread
  - performance
prerequisites:
  - Basic understanding of JavaScript
  - Access to theme functions.php or child theme
audience: intermediate
sources:
  - https://wpmudev.com/blog/delay-javascript-execution/
  - https://wp-rocket.me/blog/deferred-loading-of-javascript/
  - https://onlinemediamasters.com/minimize-main-thread-work-wordpress/
  - https://nitropack.io/blog/post/minimize-main-thread-work
  - https://onlinemediamasters.com/reduce-javascript-execution-time-wordpress/
related:
  - concept-core-web-vitals.md
  - howto-critical-css.md
---

# How to Optimize JavaScript in WordPress

JavaScript is the primary cause of poor INP (Interaction to Next Paint) scores. Optimizing JS loading and execution directly improves responsiveness and overall Lighthouse performance.

## Understanding JavaScript Loading

### Loading Strategies

| Strategy | Behavior | Use Case |
|----------|----------|----------|
| **Default** | Blocks parsing until downloaded and executed | Critical inline scripts |
| **async** | Downloads parallel, executes immediately when ready | Independent scripts (analytics) |
| **defer** | Downloads parallel, executes after DOM parsed | Scripts needing DOM |
| **delay** | Only loads on user interaction | Non-critical scripts |

### Visual Timeline

```
Default:     HTML ━━━ [JS Download] ━━━ [JS Execute] ━━━ HTML ━━━
async:       HTML ━━━━━━━━━━━━━━━━━━━━━ HTML ━━━━━━━━━━━━━━━━━
             [JS Download] [JS Exec]
defer:       HTML ━━━━━━━━━━━━━━━━━━━━━ HTML ━━ DOM Ready ━━━
             [JS Download]                    [JS Execute]
delay:       HTML ━━━ Interactive ━━━━━━━━━━━━━━━━━━━━━━━━━━━
                                [User Interaction] [JS Load + Exec]
```

## Part 1: Adding Defer/Async Attributes

### WordPress 6.3+ (Native Support)

WordPress 6.3 introduced native defer/async support:

```php
// Add defer to a script
wp_script_add_data('my-script', 'strategy', 'defer');

// Add async to a script
wp_script_add_data('my-script', 'strategy', 'async');

// When registering/enqueuing
wp_register_script(
    'my-script',
    get_template_directory_uri() . '/js/script.js',
    array(),
    '1.0',
    array(
        'strategy' => 'defer',
        'in_footer' => true
    )
);
```

### Legacy Method (Pre-6.3)

```php
function add_defer_async_attributes($tag, $handle, $src) {
    // Scripts to defer
    $defer_scripts = array('jquery-migrate', 'comment-reply', 'wp-embed');

    // Scripts to async
    $async_scripts = array('google-analytics', 'gtag');

    if (in_array($handle, $defer_scripts)) {
        return str_replace(' src', ' defer src', $tag);
    }

    if (in_array($handle, $async_scripts)) {
        return str_replace(' src', ' async src', $tag);
    }

    return $tag;
}
add_filter('script_loader_tag', 'add_defer_async_attributes', 10, 3);
```

### Defer All Scripts (with exceptions)

```php
function defer_all_scripts($tag, $handle, $src) {
    // Skip admin scripts
    if (is_admin()) {
        return $tag;
    }

    // Scripts that should NOT be deferred
    $no_defer = array('jquery-core', 'jquery');

    if (in_array($handle, $no_defer)) {
        return $tag;
    }

    // Skip if already has defer/async
    if (strpos($tag, 'defer') !== false || strpos($tag, 'async') !== false) {
        return $tag;
    }

    return str_replace(' src', ' defer src', $tag);
}
add_filter('script_loader_tag', 'defer_all_scripts', 10, 3);
```

## Part 2: Delay JavaScript Execution

Delay JavaScript completely defers loading until user interaction (mouse move, scroll, click, touch).

### Using WP Rocket

WP Rocket's "Delay JavaScript execution" feature:

1. Enable in File Optimization → JavaScript Files
2. Add scripts to delay list (or use "Delay all JavaScript")
3. Exclude critical scripts

### Manual Implementation

```php
function delay_javascript_loading() {
    if (is_admin()) return;
    ?>
    <script>
    const loadScriptsOnInteraction = () => {
        // Load all delayed scripts
        document.querySelectorAll('script[data-delay-src]').forEach(script => {
            const newScript = document.createElement('script');
            newScript.src = script.getAttribute('data-delay-src');
            if (script.getAttribute('data-delay-id')) {
                newScript.id = script.getAttribute('data-delay-id');
            }
            document.body.appendChild(newScript);
            script.remove();
        });

        // Remove event listeners after first interaction
        ['mouseover', 'click', 'keydown', 'touchstart', 'scroll'].forEach(event => {
            window.removeEventListener(event, loadScriptsOnInteraction);
        });
    };

    // Listen for user interaction
    ['mouseover', 'click', 'keydown', 'touchstart', 'scroll'].forEach(event => {
        window.addEventListener(event, loadScriptsOnInteraction, { once: true, passive: true });
    });

    // Fallback: load after 5 seconds anyway
    setTimeout(loadScriptsOnInteraction, 5000);
    </script>
    <?php
}
add_action('wp_head', 'delay_javascript_loading', 1);

// Convert scripts to delayed loading
function convert_to_delayed_script($tag, $handle) {
    $delay_scripts = array(
        'facebook-pixel',
        'google-analytics',
        'hotjar',
        'intercom'
    );

    if (in_array($handle, $delay_scripts)) {
        return preg_replace(
            '/<script(.*)src=["\']([^"\']+)["\'](.*)>/',
            '<script$1 data-delay-src="$2" $3>',
            $tag
        );
    }

    return $tag;
}
add_filter('script_loader_tag', 'convert_to_delayed_script', 20, 2);
```

## Part 3: Minimizing Main Thread Work

### Remove Unused Scripts

```php
function remove_unused_scripts() {
    // Remove WordPress emoji scripts
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('wp_print_styles', 'print_emoji_styles');
    remove_action('admin_print_scripts', 'print_emoji_detection_script');
    remove_action('admin_print_styles', 'print_emoji_styles');

    // Remove embed script (if not using embeds)
    wp_dequeue_script('wp-embed');

    // Remove jQuery Migrate (if not needed)
    if (!is_admin()) {
        wp_dequeue_script('jquery-migrate');
    }

    // Remove comment reply script (if comments disabled)
    if (!is_singular() || !comments_open()) {
        wp_dequeue_script('comment-reply');
    }

    // Remove Gutenberg block library CSS on frontend
    if (!is_admin()) {
        wp_dequeue_style('wp-block-library');
        wp_dequeue_style('wp-block-library-theme');
        wp_dequeue_style('wc-blocks-style'); // WooCommerce blocks
    }
}
add_action('wp_enqueue_scripts', 'remove_unused_scripts', 100);
```

### Conditionally Load Scripts

```php
function conditional_script_loading() {
    // Only load Contact Form 7 JS on pages with forms
    if (!is_page(array('contact', 'quote', 'booking'))) {
        wp_dequeue_script('contact-form-7');
        wp_dequeue_style('contact-form-7');
    }

    // Only load WooCommerce JS on shop pages
    if (!is_woocommerce() && !is_cart() && !is_checkout()) {
        wp_dequeue_script('wc-cart-fragments');
        wp_dequeue_script('woocommerce');
        wp_dequeue_style('woocommerce-general');
    }

    // Only load Slider Revolution on homepage
    if (!is_front_page()) {
        wp_dequeue_script('revslider');
        wp_dequeue_style('revslider');
    }
}
add_action('wp_enqueue_scripts', 'conditional_script_loading', 999);
```

### Using Asset CleanUp / Perfmatters

```bash
# Install Asset CleanUp (free)
wp plugin install wp-asset-clean-up --activate

# Or Perfmatters (premium) for more control
```

These plugins provide UI to disable scripts/styles per page.

## Part 4: Reducing JavaScript Size

### Minification

```php
// WP Rocket handles this automatically
// Or use Autoptimize:
wp option update autoptimize_js 'on'
wp option update autoptimize_js_trycatch 'on'
```

### Remove Inline Scripts

Move inline scripts to external files:

```php
// Instead of wp_add_inline_script, create external file
wp_register_script('my-config', false);
wp_enqueue_script('my-config');
wp_add_inline_script('my-config', 'const config = ' . json_encode($config) . ';');

// Better: Pass via wp_localize_script
wp_localize_script('my-script', 'MyConfig', array(
    'ajaxUrl' => admin_url('admin-ajax.php'),
    'nonce' => wp_create_nonce('my-nonce')
));
```

### Code Splitting

For custom themes with build process:

```javascript
// webpack.config.js
module.exports = {
    entry: {
        main: './src/js/main.js',
        slider: './src/js/slider.js',
        forms: './src/js/forms.js'
    },
    optimization: {
        splitChunks: {
            chunks: 'all'
        }
    }
};
```

## Part 5: Optimizing Third-Party Scripts

### Common Culprits

| Script | Impact | Solution |
|--------|--------|----------|
| Google Analytics | Medium | Use minimal snippet, delay |
| Facebook Pixel | High | Delay until interaction |
| Google Tag Manager | High | Delay, minimize containers |
| Chat widgets | Very High | Load on scroll/click only |
| YouTube embeds | Very High | Use facade/lazy load |

### Delay Analytics

```php
function defer_google_analytics() {
    ?>
    <script>
    function loadGA() {
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', 'G-XXXXXXXX');

        var s = document.createElement('script');
        s.async = true;
        s.src = 'https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX';
        document.body.appendChild(s);
    }

    // Load after 2 seconds or interaction
    setTimeout(loadGA, 2000);
    </script>
    <?php
}
add_action('wp_footer', 'defer_google_analytics');
```

### YouTube Facade (Lite YouTube Embed)

```bash
wp plugin install starter-starter-starter --activate
# Or use custom implementation:
```

```php
function youtube_facade_script() {
    ?>
    <script>
    document.querySelectorAll('.youtube-facade').forEach(el => {
        el.addEventListener('click', function() {
            const videoId = this.dataset.videoId;
            this.innerHTML = `<iframe src="https://www.youtube.com/embed/${videoId}?autoplay=1"
                allow="autoplay; encrypted-media" allowfullscreen></iframe>`;
        });
    });
    </script>
    <?php
}
add_action('wp_footer', 'youtube_facade_script');
```

### Chat Widget Delay

```javascript
// Load chat widget on scroll only
let chatLoaded = false;
window.addEventListener('scroll', function() {
    if (!chatLoaded && window.scrollY > 200) {
        // Load chat widget script
        const script = document.createElement('script');
        script.src = 'https://chat-widget.js';
        document.body.appendChild(script);
        chatLoaded = true;
    }
}, { passive: true });
```

## Part 6: Breaking Up Long Tasks

Long tasks (>50ms) block the main thread and hurt INP.

### Identify Long Tasks

```javascript
// Performance Observer for long tasks
new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        console.log('Long Task:', entry.duration, 'ms', entry.attribution);
    }
}).observe({ entryTypes: ['longtask'] });
```

### Use requestIdleCallback

```javascript
// Split heavy work into chunks
function processLargeArray(items) {
    let index = 0;

    function processChunk(deadline) {
        while (index < items.length && deadline.timeRemaining() > 0) {
            processItem(items[index]);
            index++;
        }

        if (index < items.length) {
            requestIdleCallback(processChunk);
        }
    }

    requestIdleCallback(processChunk);
}
```

### Use Web Workers

```javascript
// Offload heavy computation
const worker = new Worker('/js/heavy-task-worker.js');

worker.postMessage({ data: largeDataSet });

worker.onmessage = function(e) {
    console.log('Result:', e.data);
};

// heavy-task-worker.js
self.onmessage = function(e) {
    const result = heavyComputation(e.data);
    self.postMessage(result);
};
```

## Part 7: Plugin-Based Solutions

### WP Rocket

```bash
# Key JavaScript settings
wp option update wp_rocket_settings[defer_all_js] 1
wp option update wp_rocket_settings[delay_js] 1
```

### Perfmatters

Best for selective script unloading:
- Disable scripts per page/post
- Script Manager for granular control
- Database optimization

### Flying Scripts

```bash
wp plugin install flying-scripts --activate
```

Free plugin for delaying JavaScript until interaction.

### Async JavaScript

```bash
wp plugin install async-javascript --activate
```

UI for adding async/defer to scripts.

## Part 8: Testing and Measurement

### Chrome DevTools Performance

1. Open DevTools → Performance tab
2. Record page load
3. Look for:
   - Long yellow bars (JavaScript)
   - Tasks >50ms (highlighted)
   - Total Blocking Time

### Coverage Report

1. DevTools → More tools → Coverage
2. Reload page
3. Check JS file usage percentage
4. High unused % = optimization opportunity

### Lighthouse JavaScript Metrics

| Metric | Target |
|--------|--------|
| Total Blocking Time | <200ms |
| JavaScript Execution Time | <2s |
| Unused JavaScript | <100KB |

### Real User Monitoring

```javascript
// Measure INP
new PerformanceObserver((list) => {
    for (const entry of list.getEntries()) {
        // entry.duration is INP value
        console.log('INP:', entry.duration);
    }
}).observe({ type: 'first-input', buffered: true });
```

## Troubleshooting

### jQuery Dependency Issues

```php
// If plugins break with deferred jQuery
function move_jquery_to_footer() {
    wp_scripts()->add_data('jquery-core', 'group', 1);
    wp_scripts()->add_data('jquery-migrate', 'group', 1);
}
add_action('wp_enqueue_scripts', 'move_jquery_to_footer');
```

### Script Order Issues with defer

All deferred scripts maintain order. If issues persist:

```php
// Ensure dependency chain
wp_enqueue_script('child-script', $url, array('parent-script'), '1.0', true);
```

### Third-Party Scripts Breaking

Exclude problematic scripts from optimization:

```php
// WP Rocket exclusion
add_filter('rocket_exclude_defer_js', function($exclude) {
    $exclude[] = '/problem-script.js';
    return $exclude;
});
```

### Measuring Before/After

```bash
# Lighthouse CLI comparison
lighthouse https://site.com --preset=desktop --output=json \
  --output-path=./before.json

# Apply optimizations, then:
lighthouse https://site.com --preset=desktop --output=json \
  --output-path=./after.json

# Compare TBT, Speed Index, Total Blocking Time
```
