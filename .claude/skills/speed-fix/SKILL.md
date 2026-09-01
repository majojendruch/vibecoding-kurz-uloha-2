---
name: speed-fix
description: "Use when a WordPress or WooCommerce site failed its performance budget and you need to find the cause: slow server response, late largest contentful paint, blocked main thread, or layout shift. Routes each failing metric to the WordPress-specific things that usually cause it."
---

# Speed fix

Lighthouse says *what* is slow. This says *where to look in WordPress*.

Route by the metric that failed. Do not check everything; check the column that matches.

## Slow server response (high TTFB)

The server is slow before it sends a single byte, so nothing on the front end will fix it.

- **Autoloaded options.** Every page load reads every autoloaded row in `wp_options`. Plugins that
  were deleted often leave theirs behind. Over ~800 KB is a problem.
- **No page cache.** Without one, every visit runs the full PHP and database stack.
- **WP-Cron on page loads.** By default WordPress runs scheduled tasks during a visitor's request.
  A stuck job makes random page loads slow. Check for overdue events.
- **Slow queries.** Usually an unindexed meta query, and on WooCommerce usually in `wp_postmeta`.

## Late largest contentful paint (high LCP)

The biggest visible element takes too long to appear.

- **Unoptimised hero image.** A 2 MB JPEG where 150 KB of WebP would do.
- **Render-blocking CSS and fonts.** The browser cannot paint until they arrive.
- **High TTFB.** If the server response is also slow, fix that first; LCP cannot beat it.

## Blocked main thread (high TBT)

Too much JavaScript, almost always from plugins.

- **Plugin scripts loading everywhere.** A slider or form plugin that loads on every page instead
  of the ones that use it.
- **Third-party tags.** Analytics, chat widgets, pixels, ad scripts.
- **jQuery-era plugins** loading several libraries each.

## Layout shift (high CLS)

Things move after the page starts painting.

- **Images and iframes without width and height**, so the browser cannot reserve space.
- **Fonts swapping late**, reflowing every line of text.
- **Banners injected above content** by a cookie or promo plugin.

## Rules

- Measure before diagnosing. A guess is worth nothing next to a Lighthouse number.
- Name one cause and one fix per failing metric. A list of twenty suggestions is not a diagnosis.
- Never change a live site to test a hypothesis. Write down the change and what it should gain.
- Do not read `wp-config.php` or database dumps. They hold credentials and customer data, and no
  performance question needs them.
