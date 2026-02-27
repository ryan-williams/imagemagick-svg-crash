# ImageMagick SVG `gradientTransform` crash

ImageMagick crashes with **SIGABRT** (exit code 134) when converting any SVG containing a `gradientTransform` attribute on a `<linearGradient>` element.

## Minimal reproducer

**[`crash.svg`](crash.svg)** — triggers the crash:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
  <defs>
    <linearGradient id="g" gradientTransform="matrix(1 0 0 1 0 0)" gradientUnits="userSpaceOnUse">
      <stop offset="0" style="stop-color:red"/>
      <stop offset="1" style="stop-color:blue"/>
    </linearGradient>
  </defs>
  <rect fill="url(#g)" width="512" height="512"/>
</svg>
```

**[`ok.svg`](ok.svg)** — identical SVG without `gradientTransform` (converts fine):

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
  <defs>
    <linearGradient id="g" gradientUnits="userSpaceOnUse">
      <stop offset="0" style="stop-color:red"/>
      <stop offset="1" style="stop-color:blue"/>
    </linearGradient>
  </defs>
  <rect fill="url(#g)" width="512" height="512"/>
</svg>
```

## Reproducing

```bash
# Crashes (exit code 134 / SIGABRT)
convert crash.svg crash.png

# Works fine
convert ok.svg ok.png

# Also works fine (librsvg handles gradientTransform correctly)
rsvg-convert crash.svg -o crash-rsvg.png
```

## CI

The [GitHub Actions workflow](.github/workflows/repro.yml) demonstrates the crash on Ubuntu with the distro's ImageMagick package.

## Versions tested

- ImageMagick 7.1.2-15 (macOS arm64, Homebrew) — **crashes**
- ImageMagick 6.x (Ubuntu, apt) — **TBD via CI**
- librsvg (`rsvg-convert`) — **works fine**

## Upstream issue

<!-- TODO: link to ImageMagick/ImageMagick issue once filed -->
