# Image optimization with LQIP

## What is LQIP?


Think of it like a preview sketch that appears before the detailed painting arrives. Your readers see something right away instead of waiting for the complete image to download.

## Who this is for

This feature is designed for documentation teams that include many images, particularly those targeting readers on mobile devices or slower network connections. If your documentation has product screenshots, diagrams, or illustrations, LQIP can noticeably improve how fast the page *feels* to load.

## What LQIP does

When you import images into your documentation, LQIP automatically:

2. **Displays placeholders immediately**, Shows the fuzzy placeholder while the full image downloads
3. **Swaps in the full image**, Replaces the placeholder with the high-quality version once loading completes
4. **Reduces overall site size**, Delivers smaller, optimized image assets alongside placeholders

All of this happens transparently during your site build. You don't need to manually create placeholders or adjust your image syntax.

## How it improves your documentation

**Faster perceived performance**, Readers see content appear instantly instead of staring at blank space, making your site feel more responsive even if the actual image takes a moment to arrive.

**Better experience on slow connections**, Mobile users on 3G or poor Wi-Fi see placeholder content immediately while waiting for the full image, rather than experiencing a layout shift when the image finally loads.

**No manual work**, The optimization happens automatically during your build process. You import images normally, and LQIP handles the rest.

**Seamless integration**, Works transparently with how you already include images in your documentation using standard markdown or component syntax.

## Key capabilities


**Selective image optimization**, You can choose which images receive LQIP treatment, focusing the feature on images where the performance benefit matters most.

**Automatic processing**, Once configured, LQIP processes all eligible images without requiring individual attention during content creation.

## Boundaries and limitations

LQIP applies only to images you import as webpack assets—that is, images stored locally in your documentation project that webpack can process during the build. External images served from other URLs are not covered by this feature.


## What happens during your build

The LQIP loader processes your image files and generates two outputs for each image:

1. An optimized, production-ready version of the full-quality image
2. A tiny, low-resolution blurred placeholder that loads instantly

Both are embedded or referenced in your final documentation bundle, ready to work together in the browser.

## Related features

For additional ways to improve your documentation's performance and appearance, explore CSS Optimization and Performance Optimization.
