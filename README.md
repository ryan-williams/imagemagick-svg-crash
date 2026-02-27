# ImageMagick SVG `gradientTransform` crash

ImageMagick 7 crashes with a **double-free** (SIGABRT, exit code 134) when converting any SVG containing a `gradientTransform` attribute on a `<linearGradient>` element. ImageMagick 6 is not affected.

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
# Crashes (exit code 134 / SIGABRT, double-free)
magick crash.svg crash.png

# Works fine
magick ok.svg ok.png

# Also works fine (librsvg handles gradientTransform correctly)
rsvg-convert crash.svg -o crash-rsvg.png
```

## CI

The [GitHub Actions workflow](.github/workflows/repro.yml) demonstrates the crash with IM7 (built from source) and confirms IM6 is not affected.

**Error output from IM7:**
```
free(): double free detected in tcache 2
Aborted (core dumped)
```

## Versions tested

| Version | Source | Result |
|---|---|---|
| ImageMagick 7.1.2-16 (Beta) | GHA source build | **crashes** (double-free) |
| ImageMagick 7.1.2-15 | macOS arm64, Homebrew | **crashes** |
| ImageMagick 6.9.12-98 | Ubuntu apt | works fine |
| librsvg 2.58.0 (`rsvg-convert`) | Ubuntu apt | works fine |

## Upstream issue

https://github.com/ImageMagick/ImageMagick/issues/8582
