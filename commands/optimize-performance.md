---
allowed-tools: Read, Edit, MultiEdit, Bash, Grep, Glob, TodoWrite, Task
argument-hint: [optional: specific pages or directories to optimize]
description: Optimize page loading speed while preserving functionality
---

For all pages, make the page loading speed faster as possible with preserving existing style and functionality.

## Performance Optimization Strategy:

1. **Analyze current performance**
   - Identify performance bottlenecks
   - Check bundle sizes with `npm run build` or equivalent
   - Look for large dependencies in package.json
   - Find unoptimized images and assets
   - Detect render-blocking resources

2. **JavaScript optimization**
   - **Code splitting**: Split large bundles into smaller chunks
   - **Lazy loading**: Implement dynamic imports for routes/components
   - **Tree shaking**: Remove unused code and dead imports
   - **Minification**: Ensure production builds are minified
   - **Bundle analysis**: Remove duplicate dependencies
   - Example implementations:
     ```javascript
     // Before
     import HeavyComponent from './HeavyComponent';
     
     // After - Lazy load
     const HeavyComponent = lazy(() => import('./HeavyComponent'));
     ```

3. **CSS optimization**
   - **Remove unused CSS**: Use PurgeCSS or similar tools
   - **Critical CSS**: Inline critical above-the-fold styles
   - **Minify CSS**: Compress stylesheets
   - **Optimize CSS delivery**: Load non-critical CSS asynchronously
   - **Reduce specificity**: Simplify selectors where possible

4. **Image optimization**
   - **Modern formats**: Convert images to WebP/AVIF
   - **Lazy loading**: Add loading="lazy" to images below fold
   - **Responsive images**: Use srcset for different screen sizes
   - **Image compression**: Optimize file sizes without quality loss
   - **Next.js Image**: Use next/image component if applicable
   - Example:
     ```html
     <!-- Before -->
     <img src="large-image.jpg">
     
     <!-- After -->
     <img 
       src="image-optimized.webp" 
       loading="lazy"
       srcset="image-320w.webp 320w, image-640w.webp 640w"
       sizes="(max-width: 320px) 280px, 640px"
     >
     ```

5. **Network optimization**
   - **Enable compression**: Gzip/Brotli compression
   - **HTTP/2 Push**: Push critical resources
   - **CDN usage**: Serve static assets from CDN
   - **Browser caching**: Set appropriate cache headers
   - **Preload/Prefetch**: Preload critical resources
   - Add to HTML head:
     ```html
     <link rel="preload" href="critical.css" as="style">
     <link rel="prefetch" href="next-page.js">
     <link rel="preconnect" href="https://api.example.com">
     ```

6. **React/Next.js specific optimizations**
   - **Memoization**: Use React.memo, useMemo, useCallback
   - **Virtual scrolling**: For long lists
   - **Suspense boundaries**: Better loading states
   - **Server components**: Use RSC where applicable (Next.js 13+)
   - **Static generation**: Use SSG over SSR when possible
   - Example:
     ```javascript
     // Memoize expensive computations
     const expensiveValue = useMemo(() => 
       computeExpensiveValue(props), [props.dependency]
     );
     
     // Memoize callbacks
     const handleClick = useCallback(() => {
       // handler
     }, [dependency]);
     ```

7. **Bundle size reduction**
   - **Replace heavy libraries**: Find lighter alternatives
   - **Dynamic imports**: Load libraries only when needed
   - **Externalize large dependencies**: Use CDN for common libraries
   - Common replacements:
     * moment.js → day.js or date-fns
     * lodash → lodash-es with tree shaking
     * axios → native fetch

8. **Rendering optimization**
   - **Reduce DOM nodes**: Simplify component structure
   - **Debounce/throttle**: Limit expensive operations
   - **Web Workers**: Move heavy computations off main thread
   - **Request Animation Frame**: For smooth animations
   - **Intersection Observer**: For visibility-based loading

9. **Build configuration**
   - Update webpack/vite/build config:
     ```javascript
     // webpack.config.js example
     optimization: {
       splitChunks: {
         chunks: 'all',
         cacheGroups: {
           vendor: {
             test: /[\\/]node_modules[\\/]/,
             priority: 10
           }
         }
       },
       usedExports: true,
       sideEffects: false
     }
     ```

10. **Verify optimizations**
    - Test page load times before and after
    - Ensure no functionality is broken
    - Check that styles remain intact
    - Validate lazy-loaded content works
    - Monitor bundle size reduction

Target area (if specified): $ARGUMENTS