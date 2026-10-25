<p align="right"><a href="README.md">한국어</a> · <b>English</b> · <a href="README.ja.md">日本語</a> · <a href="README.zh.md">中文</a> · <a href="README.fr.md">Français</a> · <a href="README.es.md">Español</a></p>

# 🔍 Upscaly: Lossless-Oriented Upscaler (무손실 지향 확대기) `v1.2.2`

An image upscaler that aims to be lossless. It runs as a single file (`upscaler.html`) and does all of its computation inside the browser.
There is nothing to install and nothing to upload. Your images never leave your device.

Current version **v1.2.2** · what changed is in [12. Changelog](#12-changelog). Development sources, tests and research material are in the [Upscaly-dev](https://github.com/Velvitty/Upscaly-dev) repository.

**Contents**

1. [At a glance](#1-at-a-glance)
2. [Principles](#2-principles)
3. [How it works](#3-how-it-works)
4. [Algorithms and formulas](#4-algorithms-and-formulas)
5. [The physical world](#5-the-physical-world)
6. [System](#6-system)
7. [Issues and fixes](#7-issues-and-fixes)
8. [Optimization](#8-optimization)
9. [Validation](#9-validation)
10. [Strengths](#10-strengths)
11. [Limitations](#11-limitations)
12. [Changelog](#12-changelog)
13. [References](#13-references)

---

## 1. At a glance

### What is this tool?

When you enlarge a small image, it usually turns blurry, gets jagged staircases, or grows white rims around its edges. Upscaly enlarges different kinds of images — photos, illustrations, comics, documents, logos, pixel art — each in the way that suits it, reducing these flaws while preserving as much of the information the original image contained as possible.

"Aiming to be lossless" means the restoration is fitted so that shrinking the enlarged image back to its original size gives the original again. The enlargement does not invent picture content that was not in the original; it concentrates on laying out the information the original has in the most plausible way. Corrections layered on top, such as edge enhancement or staircase removal, are steps that refine this restoration to look better to the eye, so turning them on moves the result slightly away from the original information ([2.1](#21-upscaling-does-not-create-information)).

### Why it is good

- **Nothing to install.** Open one file in a browser and you are done. No internet connection needed.
- **Your images stay with you.** It uses no server and no external libraries.
- **It tunes itself.** The 「Auto」(자동) button identifies the kind of image among 9 categories and sets the settings by measuring that image directly.
- **It holds up at large sizes.** The image is computed in tiles, and each row of tiles is compressed and appended, so even if the result is hundreds of millions of pixels, the whole uncompressed result is never held in memory. The compressed PNG file, however, accumulates in memory until saving finishes.
- **Band-free gradients.** When saving at 8 bits, fine dither is mixed in so that heavily enlarged skies and gradients do not show stair-like bands. You can also save as 16-bit PNG.
- **Phone photo colors kept intact.** Display P3 photos from iPhones and similar devices are read as P3 and saved as P3, so vivid reds and greens do not turn dull.
- **Texture revival (optional).** You can put fine grain back onto skin and fabric that became waxy-smooth from heavy enlargement. It is off by default.
- **Transparent images stay clean too.** Colors do not bleed at semi-transparent edges.
- **Two goals: accuracy and perception.** The default 「Accuracy」(정확도) upscales as close to the original as possible. 「Perceptual」(체감) makes edges steeper and brings out buried contrast, so the result looks crisper to the eye. No white fringes (halos) appear.

### How to use

1. Open `upscaler.html` in a modern browser.
2. Press **Open image** (이미지 열기), drag an image onto the screen, or paste one. You can open up to 20 images at once, in JPEG, PNG, WebP or AVIF.
3. Choose the **scale** (배율). There is a width/height-based scale (1–12×) and a pixel-count (area) based scale; choosing 「Auto」 picks the largest scale (3–8×) this device can process to the end.
4. Press **「Auto: (recommended, suggested)」(자동: (권장, 추천))**. It measures the kind and condition of the image, fills in all the settings below, and writes a short note on what it based them on. If looking crisp matters more than staying faithful to the original, choose 「Perceptual」 under **Goal** (목표).
5. Press **Run upscale** (확대 실행). With **Compare original** (원본 비교) on, the left half of the screen shows the original enlarged by the browser and the right half shows the result. Drag to pan, use the wheel to zoom, and use the **Fit** (맞춤, F key), **1:1** and **Compare original** (C key) buttons.
6. **Save PNG** (PNG 저장) saves the current image, and **Save all** (모두 저장) saves all processed images at once.

### Choosing settings yourself

The panels on the right side of the screen are laid out in processing order.

| Panel | What it does |
|---|---|
| Scale (배율) | How many times to enlarge. 1× only cleans up without enlarging |
| Preset · Auto (프리셋 · 자동) | 33 presets or 「Auto」 |
| Goal (목표) | 「Accuracy」 (closest to the original, default) or 「Perceptual」 (raises edge steepness and local contrast to look crisper, strength 25–150%). Presets and 「Auto」 do not change it |
| Reconstruction kernel (재구성 커널) | How to fill in new pixels (DCCI · pixel art · Lanczos · Catmull–Rom · Mitchell · nearest), staged upscaling, ringing suppression, source consistency |
| Noise · artifact cleanup (노이즈·잡티 정리) | Removes noise and JPEG artifacts before enlarging |
| Sharpness restoration (선명도 복원) | Undoes blur from shooting or scanning before enlarging |
| Staircase removal (계단 제거) | Averages only along the edge direction to remove staircases, contour redrawing, JPEG deblocking |
| Edge enhancement (윤곽 강조) | Unsharp mask with halo suppression |
| Texture revival (질감 되살리기) | Adds fine grain that was not in the original. Off by default, 20–30% recommended for photos and paintings |
| Color processing (색 처리) | White balance (a separate feature that presets and Auto do not touch), linear-light computation, 8-bit dithering, 16-bit saving |

Checkboxes and sliders work exactly as you set them. For example, 「Upscale in stages」(단계로 나눠 확대), when on, splits the upscale even if measurements say it is worse. Instead, the indicator next to it tells you the result measured on the current image. Right after an image is opened, the initial value of this checkbox is decided by measuring the image; if you change the checkbox or press 「Auto」 before that measurement finishes, the late-finishing measurement does not touch your choice.

### Good to know

- An image that has already been shrunk and re-saved many times has lost the detail it originally had, and that detail cannot be brought back. Upscaly only lays out the remaining information in the most plausible way; it does not invent faces or letters that are not there.
- For logos and drawings that have a vector original (SVG, AI), using that original is always better.
- Enlarging 8× produces a result of hundreds of megapixels and takes a long time. The progress display shows the remaining time.

---

## 2. Principles

### 2.1 Upscaling does not create information

A digital image is a set of samples taken from a continuous scene at regular intervals. Patterns finer than the sampling interval were never recorded in the first place. So the essence of upscaling is "how to fill in between the samples", and depending on how you fill them in, flaws such as blur, staircases and ringing appear.

Upscaly follows three principles when choosing how to fill them in. The first two are constraints of the restoration stage (interpolation → cell range → consistency) and hold when the corresponding checkboxes (「Block ringing · halos」(링잉·후광 차단), 「Source consistency correction」(원본 정합 보정)) are on.

1. **Create no new extremes.** A restored pixel lies between the darkest and brightest values of the original 2×2 cell it belongs to (cell range limit). Ringing and halos are extremes that did not exist in the original, so they do not arise at this stage.
2. **Shrinking back gives the original.** The restored image is fitted so that shrinking it back by area-averaging light gives the original (source consistency correction). This means it does not contradict the information in the original.
3. **Fill edges along their direction.** Mixing across an edge blurs it, and mixing along the axes creates staircases. The direction in which an edge runs is read, and filling happens only along that direction (DCCI).

The two constraints (1 · 2) are hard to satisfy at the same time. Satisfying one slightly breaks the other. So alternating projection, which satisfies them in turn, is used ([4.6](#46-alternating-projection)).

Unsharp masking, directional smoothing, contour redrawing and texture revival after restoration are steps that refine the result for the eye, and they do not re-enforce these two constraints. So a final result with them on may slightly exceed the original cell ranges, or differ slightly from the original when shrunk back. The on-screen "upscale consistency error" is measured on values before these corrections. Presets such as 「Minimal intervention · pure Lanczos」(최소 개입 · 순수 Lanczos) turn both constraints off.

### 2.2 Light mixes in linear light

The values stored for the screen (sRGB) are bent to match the human eye. Mixing the light of two pixels is correct in the actual amount of light (linear light), not in these bent values. Interpolating directly on the bent values makes the boundary between bright and dark areas clump darker. Upscaly computes in linear light by default, and converts back to sRGB only when saving. Turning off the 「Compute in linear light」(선형광에서 계산) checkbox computes directly in gamma space (not recommended).

Transparent images are computed with color multiplied by alpha (premultiplied). And in source preparation, in the intermediate stages of staged upscaling, and in the final tile computation alike, **alpha goes through exactly the same operations as color.** If color and alpha were filtered differently, dividing color by alpha later would distort the color of semi-transparent edges.

### 2.3 Blur and noise are handled before upscaling, at the original resolution

Camera lenses and scanners capture a scene slightly blurred, and sensors and compression add noise. These flaws arose at the original resolution, so it is right to undo them at the original resolution. Handling them after upscaling means the flaws have been enlarged too, which is harder and slower.

- **Blur** is inverted with Richardson–Lucy deconvolution. Most of the reason the upscaled image looks crisper than the original comes from here.
- **Noise** is removed based on a noise spectrum measured directly from the image. Components larger than the noise (edges, texture) are kept, and only components indistinguishable from noise are reduced.
- **JPEG artifacts** are removed by reading the file's quantization tables. Here the luminance DCT coefficients are brought back so that they do not leave the quantization intervals of the original file, so detail recorded in the file is not erased. This guarantee applies only to luminance in this projection step. The chroma components, and the deblocking, noise cleanup, deconvolution, upscaling and sharpening that follow, can move coefficients outside their intervals.

### 2.4 Lines are redrawn

In graphics where two colors split sharply (logos, line art, text), the original's staircase itself tends to be preserved as if it were the "correct answer". Upscaly finds such boundaries and redraws them at the upscaled resolution as contours one pixel wide. It takes the 50% isoline of the smooth interpolated values as the contour position and builds the profile according to that distance (signed-distance method). Directional smoothing straightens the shape of the contour and redrawing sharpens its profile, so using both together gives lines that are smooth without staircases and do not bleed.

### 2.5 Invented texture is grain, not texture

Upscaling cannot create patterns finer than the original pixel spacing. So in a heavily enlarged photo, skin texture, fabric threads and leaf veins disappear and it looks waxy-smooth. The **amount** of the lost texture can be estimated. This is because natural images hold similar energy each time the spatial frequency doubles (the power spectrum is roughly $`1/f^2`$). From the strength of texture measured in the original's fine band, one can estimate how much texture the missing octave would have had.

But the **position and shape** of the texture are not in the original. What Upscaly adds is noise matched to that amount, that is, fine grain like film grain. Since its positions do not match the ground truth, the more the energy is filled up to the ground truth, the larger the difference from the original (PSNR) becomes, and to the eye it starts to look more like noise than texture. That is why texture revival is an optional feature that 「Auto」 does not touch, and the guidance is to use it at low strength on smooth surfaces such as portraits, skin and fabric. Drawing a plausible texture shape as well would require a generative model (neural network) ([11. Limitations](#11-limitations)).

### 2.6 8-bit staircases are hidden with fine noise

Processing is done in floating point, but the commonly saved 8-bit PNG records brightness in only 256 levels. Enlarging a smooth sky 8× turns brightness that changed by one level every few pixels in the original into bands tens of pixels wide. Mixing in noise smaller than half a level just before rounding (dithering) scatters the band boundaries into fine grain that is invisible to the eye. The noise is a pattern determined only by screen coordinates, so it is the same for every tile, and it is applied equally to the three channels so gray stays gray. Values that already fall exactly on a level, such as flat solid areas, do not change.

### 2.7 Phone photo colors are wider than sRGB

Recent phone photos are often captured in the Display P3 gamut, which is wider than sRGB. Reading them into an ordinary (sRGB) canvas makes the browser squeeze the vivid reds and greens outside sRGB into sRGB, so the upscaled result looks duller than the original. Upscaly reads the color profile of the original file, reads it into a P3 canvas if the gamut is wide, and saves the result with a P3 profile attached. Display P3 uses the same transfer curve as sRGB, so linear-light computation stays the same; only the primaries differ.

### 2.8 Each image has a different answer

Settings that suit photos ruin line art, and settings that suit line art ruin photos. Source consistency is good for photos, but in line art where two colors split cleanly it restores the original's staircase as the correct answer. So 「Auto」 first identifies the kind of image (9-class classifier), starts from the preset suited to that kind, and adjusts details with values measured on that image. For large enlargements it uses values found by searching a space of 1.52 trillion settings ([9.4](#94-large-scale-restoration-search)).

### 2.9 Accuracy and perceived sharpness are different goals

The result closest to the original (the one with the highest PSNR) is not the one that looks sharpest. Upscaling blunts even the edges the source resolution can hold (edge slope 0.81× the ground truth) and weakens contrast at the scale of a few source pixels (0.94×). The eye reads both as sharpness. So there are two goals. 「Accuracy」 is the restoration as before; 「Perceptual」 brings these two back to the ground-truth level on top of it. It creates no new extrema, so no white fringes appear, and it does not draw in texture the source never had. In exchange, the numbers against the original drop slightly ([4.21](#421-perceived-sharpness), [9.10](#910-perceived-sharpness)).

---

## 3. How it works

### 3.1 Overall flow

```
File ─▶ decode (EXIF orientation) ─▶ color profile (P3 canvas if wide gamut) ─▶ JPEG header (quantization tables · sampling) ─▶ subject box
                                              │
              「Auto」: analysis image ─▶ 9-class classification ─▶ preset + measured adjustments ─▶ large-scale restoration search values / directional smoothing measurement / staged upscaling measurement
                                              │
Source preparation (original resolution) : linear-light planes (premultiplied) ─▶ color processing ─▶ precise DCT restoration ─▶ deblocking ─▶ noise cleanup ─▶ RL deconvolution ─▶ staged upscaling intermediate stages
                                              │
Tile computation (worker pool)   : interpolation ─▶ cell range ─▶ consistency↔cell range alternating projection ─▶ statistics ─▶ unsharp ─▶ directional smoothing ─▶ contour redrawing ─▶ (texture) ─▶ un-premultiply ─▶ encoding (8-bit dither)
                                              │
                         Tile cache (LRU) ─▶ screen / PNG saving per tile row (8 · 16 bits, sRGB or P3)
```

### 3.2 Loading

A file is first decoded with `createImageBitmap` (applying EXIF orientation); if that is blocked, it falls back to a Blob URL in `<img>`, and then to a Data URL. Wide-gamut files (section 2.7) are detected before decoding and use the `<img>` path first (Blob URL, Data URL if blocked), falling back to `createImageBitmap` only when both fail. This is because the default color conversion of `createImageBitmap` differs between browsers and can clip to sRGB. Browsers can really take tens of seconds on large files, so the time limit is set in proportion to file size (45 s + 9 s per MB, at most 5 minutes). No path is ever left waiting without end.

For JPEG, only the header is read to get the quantization tables, the sampling ratio (4:2:0 etc.), an estimated quality and the EXIF orientation. The entropy-coded data is not decoded. Whether the decoder applied the EXIF orientation is checked from the width and height of the decoded result, and the JPEG 8×8 grid is realigned to the display orientation.

The color profile is also looked up in the file (it follows fragment, chunk and box headers and reads only the parts needed, so it is found even if large metadata or image boxes come first). For JPEG it is the APP2 `ICC_PROFILE` fragments, for PNG the cICP · iCCP · sRGB chunks (in this priority regardless of order), for WebP the ICCP chunk, and for AVIF · HEIF the colr box associated with the primary image. If the gamut is wider than sRGB, the source canvas is created as `display-p3` and drawn into, and the info line says "Display P3". If the browser does not support P3 canvases, it reads as sRGB as before.

For each image, the original is shrunk to about 360px, blobs of saliency (distance from the mean color) are found, and a subject box is made. It can be viewed with the 「Subject」(피사체) button on screen.

### 3.3 Source preparation (`prepareSource`)

All corrections that complete at the original resolution happen here. The results are cached, together with a key of the settings, for up to the two most recent, so even if an upscaled result is evicted from the cache, the same settings reuse it.

1. **Linear-light planes.** 8-bit sRGB is converted to linear light with a 256-entry table, and if there is alpha, color is multiplied by alpha.
2. **Color processing.** Only when the white balance strength is not 0 (default 0).
3. **Precise DCT restoration.** The original is cleaned up using the noise spectrum and the JPEG quantization tables. It is split into bands handed to workers, and the DCT grid is fixed to global coordinates, so there are no seams at band boundaries.
4. **JPEG deblocking.** Steps at multiples of 8 are flattened only when they are smaller than the surrounding variation.
5. **Noise cleanup.** Only neighbors of similar brightness are averaged (bilateral filter). Alpha is averaged with the same weights.
6. **RL deconvolution.** Alpha goes through the same number of iterations.
7. **Intermediate stages of staged upscaling.** If a large scale is split into two or more stages, everything up to the last stage is enlarged here. DCCI uses directional interpolation in intermediate stages too, and pixel art restores contours at each stage. Alpha goes through the same direction weights, cell range and inter-stage sharpening as color. The reference for the cell range and consistency of the final stage is the **original**, not the intermediate image.

### 3.4 Tile computation (`renderTile`)

The result is computed in tiles (512–1024px depending on scale, a multiple of the scale for integer scales). Each tile receives the original patch it needs (interpolation support + consistency margin + margins for the direction field and contour model) and is computed independently, so the result is the same whichever tile is computed first.

1. **Interpolation.** With DCCI, all channels are enlarged at once (`dcciStageMulti`). The direction weights are computed only once on a guide plane (gamma luminance + an alpha edge term) and shared between channels. Then Catmull–Rom brings it to the exact scale.
2. **Cell range limit.** Values are clipped to the per-channel minimum and maximum of the original 2×2 cell.
3. **Alternating projection of consistency correction and cell range.** The two are projected alternately twice (once for consistency if cell range is off). Each pass is solved on a region 7 pixels wider than the result region, so no pass reads outside the coefficient grid.
4. **Statistics.** To show on screen how much the result differs from the original when shrunk back (upscale consistency error), it is measured on values before the corrections.
5. **Unsharp.** Halos are suppressed with a knee limiter proportional to local amplitude.
6. **Directional smoothing.** Averaging only along the tangent direction found from the original's structure tensor.
7. **Contour redrawing.** Contours one pixel wide are redrawn by signed distance.
8. **Perceived sharpness.** Only when the goal is 「Perceptual」, raises edge slope and local contrast in the luminance. It never exceeds the surrounding brightness range ([4.21](#421-perceived-sharpness)).
9. **Texture revival.** Only when on, fine texture missing from the result is added in proportion to the texture strength measured on the original. With staged upscaling it is measured on the original, not on the intermediate image. It is not used with the nearest kernel, which copies the original as is, or the pixel-art kernel, which preserves the palette, and the slider is turned off too.
10. **Un-premultiplication and encoding.** Converted to 8 bits for the screen and 8 · 16 bits for saving. For 8 bits, with dithering on (default), fine noise determined by coordinates is mixed in just before rounding. The screen and 8-bit saving use the same tiles, so what you see is what is saved.

The worker pool starts cores − 1 workers (at most 8), and if they do not respond within 5 seconds, the same engine computes on the main thread. The engine (`ENGINE_FACTORY`) is a bundle of pure computation functions, so the same code is also loaded as a Blob URL worker.

### 3.5 Staged upscaling

A method of enlarging a large scale over several steps. You choose among 2× first, large scale first, balanced (smaller or larger first), even (√×√), and three stages. Sharpening can be applied between stages. The indicator shrinks the center of the current image by the target scale with area-averaging of light, then compares a single-step upscale and an upscale split in the current way against the original (PSNR), and tells you whether the current checkbox state is the better one. The result is remembered per image and setting.

### 3.6 Auto

1. **Analysis.** If the long side exceeds 1024, the image is halved with 2×2 averaging, then shrunk with integer-factor box averaging to 512 or less, and 27 features are measured on this analysis image. These include the structure tensor (texture ratio, axis-aligned edges), strong edges and contour lines, text-line periodicity, long straight lines, color and brightness composition, 8×8 block flat residual on an original-resolution patch, a nearest-neighbor upscaling grid, the number of original colors, screentone and so on.
2. **Classification.** Multinomial logistic regression picks one of photo, illustration, painting, comic, line art, document, UI, logo and pixel art. Pixel art is confirmed by structure (a nearest-neighbor integer grid, or a small palette at 1×) rather than by probability. The reasons that pushed the decision (per-feature contributions to the logit difference between first and second place) are written on screen.
3. **Preset and measured adjustments.** Starting from the preset suited to the kind, deconvolution iterations · σ, JPEG restoration, deblocking and so on are adjusted by noise (median Laplacian of flat areas), edge width, 8px grid contrast and JPEG quality.
4. **Large-scale restoration search values.** If the kind and scale meet the validation conditions, the settings found by search are used as is (`AUTO_BIG`). Photos, line art, UI and paintings at 2× or more, documents at 4× or more. Logos and comics use them only at 2× or more and when the file is JPEG, because they did not pass validation on clean input ([9.4](#94-large-scale-restoration-search)). Illustrations do not use them.
5. **Per-image measurement.** When search values are not used, for graphics (excluding photos and pixel art) the strength of directional smoothing is chosen by measuring it on that image (`autoAA`), and then all staged-upscaling methods are measured and the best is chosen (`autoStage`).

Auto requests are numbered, so an earlier measurement that finishes late is not applied to a newly opened image. If you change the scale after Auto, it tells you to press it again.

### 3.7 Auto scale

Among 3–8×, it picks the largest scale whose rendering is **guaranteed**. Pressing 「Auto」 in the scale row moves the scale at once to the largest one guaranteed with the current settings. Guaranteed means the peak memory over the whole process is within 80% of the memory budget, the remaining share is larger than the minimum cache, and the result is within the save limit. It also counts the compressed PNG that accumulates in memory while saving (maximum compression ratio measured on 47 images in the data, plus a 10% margin) and the prepared originals of previous images still in the cache. Since the settings 「Auto」 picks (preset · large-scale restoration search values · staged upscaling) differ by scale, the budget differs too, so 「Auto」 actually applies each scale's settings starting from the largest scale and picks the first guaranteed scale. When staged upscaling is decided by measuring this photo, only methods within the guarantee are chosen. Once chosen, the tile cache limit is actually lowered to that value. If even 3× is not guaranteed, it falls back to the largest guaranteed scale below that.

### 3.8 Saving

If the browser supports `CompressionStream('deflate')`, each row of tiles (128-row bands for 16 bits) is compressed and appended as PNG fragments. 8 bits reuses the tiles in the cache, and 16 bits recomputes each band. No large canvas holding the whole result is created, but the compressed fragments are collected in memory until saving finishes and then exported as one file. The file contains a color space chunk and a `Software` iTXt chunk (UTF-8) recording the kernel and scale. The color space chunk is normally sRGB; for images read as P3, it is cICP (primaries 12 = Display P3, transfer 13 = sRGB curve) and a Display P3 ICC profile (iCCP) with the same meaning. Programs that read cICP use that, and programs that do not use the ICC. Browsers without streaming compression can save only 8 bits. They collect the result in one canvas and save with the browser encoder, and if the canvas limit is exceeded, they learn that device's actual limit. Choosing 16-bit saving in such browsers makes saving fail.

### 3.9 Color processing

Color processing is a separate feature from upscaling and defaults to 0. Pressing 「Color auto」(색 자동) estimates the illuminant color in three ways (mean, edges, bright areas) and sets the white balance strength only as far as all three agree. A real illuminant cast tints even the bright areas so all three grow together, whereas a subject's own colors, such as an orange cat or a sunset, pull only the mean. Colors in illustrations, paintings, UI, logos and pixel art are treated as intended colors and not corrected. Overall brightness is preserved and channel gains are limited to 0.55–1.8.

---

## 4. Algorithms and formulas

Below, $`I`$ is the original (linear light, premultiplied), $`U`$ is the result, and $`s_x, s_y`$ are the horizontal and vertical scales.

### 4.1 Color space

```math
\mathrm{lin}(u) =
\begin{cases}
u / 12.92 & u \le 0.04045 \\[2pt]
\left(\dfrac{u + 0.055}{1.055}\right)^{2.4} & \text{otherwise}
\end{cases}
\qquad
\mathrm{srgb}(v) =
\begin{cases}
12.92\,v & v \le 0.0031308 \\[2pt]
1.055\,v^{1/2.4} - 0.055 & \text{otherwise}
\end{cases}
```

8 bits → linear light uses a 256-entry table, and linear light → 8 bits uses an 8,192-entry table (quantization error under 1 level). The 16-bit encoder builds its table on a $`\sqrt{v}`$ axis, so it stays within 1 level even in the steep region near black. Converting the 256 levels of 8 bits to linear light and back leaves all 256 levels unchanged ([9.1](#91-invariants)).

With alpha, computation uses premultiplied values $`(\alpha R, \alpha G, \alpha B, \alpha)`$, and just before encoding, values are divided by $`\alpha`$ if $`\alpha > 10^{-7}`$.

### 4.2 Reconstruction kernels

Applied separably (horizontal → vertical), with pixel centers aligned ($`u = (X + 0.5)/s - 0.5`$). When downscaling, the kernel is stretched by the scale to prevent aliasing.

```math
L_a(x) = \mathrm{sinc}(x)\,\mathrm{sinc}(x/a)\ \ (|x| < a),\qquad
k_{B,C}(x) = \tfrac16
\begin{cases}
(12 - 9B - 6C)|x|^3 + (-18 + 12B + 6C)|x|^2 + (6 - 2B) & |x| < 1 \\
(-B - 6C)|x|^3 + (6B + 30C)|x|^2 + (-12B - 48C)|x| + (8B + 24C) & 1 \le |x| < 2
\end{cases}
```

Lanczos-3 · 4 use $`a = 3, 4`$, Catmull–Rom is $`(B, C) = (0, 0.5)`$, and Mitchell–Netravali is $`(1/3, 1/3)`$. Nearest copies the original as is at integer scales, so shrinking back gives a bit-for-bit identical original.

### 4.3 Edge-adaptive DCCI

Based on the directional cubic convolution interpolation of Zhou et al. (2012), each stage doubles the size (corner-aligned: output $`(2i, 2j)`$ = input $`(i, j)`$).

**Stage 1: the center of a 2×2 cell.** The variation along the two diagonal directions is measured over a 4×4 neighborhood. $`G`$ is the guide plane (gamma luminance 0–255) and $`n_{\mathrm{off}}`$ is the noise floor.

```math
d_1 = \sum_{(p,q) \in \nearrow} |G_p - G_q| + n_{\mathrm{off}},\qquad
d_2 = \sum_{(p,q) \in \searrow} |G_p - G_q| + n_{\mathrm{off}}
```

Each sum is over the 9 pairs adjacent along that diagonal direction. The weights are as follows ($`T = 1.15`$).

```math
(w_a, w_b) =
\begin{cases}
(0, 1) & 1 + d_1 > T(1 + d_2) \\
(1, 0) & 1 + d_2 > T(1 + d_1) \\
\left(\dfrac{a}{a+b}, \dfrac{b}{a+b}\right),\ a = \dfrac{1}{1 + d_1^{5}},\ b = \dfrac{1}{1 + d_2^{5}} & \text{otherwise}
\end{cases}
```

If the $`\nearrow`$ variation is large, the edge runs in the $`\searrow`$ direction, so interpolation is done in that direction. The value in each direction uses the cubic convolution taps $`(-1, 9, 9, -1)/16`$.

```math
p_a = \tfrac{1}{16}\big(9(f_{12} + f_{21}) - (f_{03} + f_{30})\big),\quad
p_b = \tfrac{1}{16}\big(9(f_{11} + f_{22}) - (f_{00} + f_{33})\big),\quad
v = \mathrm{clip}\big(w_a p_a + w_b p_b,\ \min_4 f,\ \max_4 f\big)
```

The final `clip` is an addition not in the original paper. It keeps the new value from exceeding the range of the 4 nearest neighbors, so ringing and halos do not arise from this stage on.

**Stage 2: pixels between horizontal and vertical neighbors.** The same procedure is applied in the horizontal and vertical directions on a grid rotated by 45°. Borders are folded by mirror reflection to preserve even symmetry.

**Number of stages.** A table measured and fixed per scale (`EDI_PLAN`) is used. At 2× or less, one stage on a 2× grid; above that, it goes up to a 4× grid (two stages) and then Catmull–Rom brings it to the exact scale ([8. Optimization](#8-optimization)).

### 4.4 Cell range limit

An output sample $`U(X, Y)`$ is clipped to the per-channel range of the original cell $`(i..i{+}1,\ j..j{+}1)`$ it belongs to.

```math
U(X, Y) \leftarrow \mathrm{clip}\Big(U(X, Y),\ \min_{(p,q) \in \mathrm{cell}} I_{p,q},\ \max_{(p,q) \in \mathrm{cell}} I_{p,q}\Big)
```

Nearest-neighbor upscaling always lies within this range, so a solution exists. The per-cell minimum and maximum are built only once per channel plane (`cellRange` cache).

### 4.5 Source consistency correction

If $`D`$ is the operation that shrinks the result back by area averaging, the goal is $`D(U) = I`$. Coefficients $`c`$ at the original resolution are spread by bilinear interpolation $`P`$ and added to the initial estimate $`U_0`$.

```math
U = U_0 + P(c),\qquad D(U_0 + P c) = I \;\Longrightarrow\; K c = I - D(U_0),\quad K = D P
```

Since $`D`$ and $`P`$ are both separable by axis, $`K = K_x \otimes K_y`$, and the $`K`$ of each axis is a tridiagonal matrix. So $`c`$ is solved exactly at the original resolution with the Thomas algorithm. The per-axis coefficients are built from each output pixel's footprint on the original (including fractional overlaps). The inverse decays by about 0.17× per pixel, so influence beyond 7 pixels is at most $`0.17^7 \approx 4 \times 10^{-6}`$. So solving each tile with a 7-pixel margin is the same as solving the whole at once.

### 4.6 Alternating projection

The consistency condition $`\{U : D U = I\}`$ is an affine set, and the cell range $`\{U : m \le U \le M\}`$ is a box set. Both are convex, so projecting alternately approaches their intersection.

```math
U^{(k+1)} = \Pi_{\mathrm{box}}\Big(\Pi_{\mathrm{affine}}\big(U^{(k)}\big)\Big),\qquad k = 0, 1
```

Upscaly projects twice (once for consistency if cell range is off). The consistency projection is not a strict orthogonal projection but a correction along the $`P`$ direction, and the last projection is the cell range, so the promise of no new extremes is always kept.

### 4.7 Richardson–Lucy deconvolution

Blur is modeled as a Gaussian $`h`$ with standard deviation $`\sigma`$, and the estimate $`e`$ is updated multiplicatively from the observation $`o`$.

```math
e^{(t+1)} = e^{(t)} \cdot \left( h \ast \mathrm{clip}\!\left(\frac{o}{h \ast e^{(t)}}\right) \right)
```

The ratio is clipped so it does not diverge. Multiplicative updates preserve constant factors, so applying the same number of iterations to premultiplied color and alpha keeps the color/alpha ratio unchanged within regions of the same color.

### 4.8 Noise cleanup (bilateral filter)

Applied separately horizontally and vertically, and the brightness-difference weights are built as a 1,025-entry table to avoid calls to the exponential function.

```math
\hat I_p = \frac{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)\, I_q}{\sum_q w_s(\|p - q\|)\, w_r(|G_p - G_q|)}
```

$`G`$ is a gamma-approximated brightness. The alpha plane is averaged with the same weights.

### 4.9 Precise DCT restoration

**Noise model.** For each channel (Y, Cb, Cr), noise is represented by a per-frequency variance $`S_c[k]`$ of the 8×8 DCT and a gain per brightness bin $`g_c(I)`$.

1. Blocks with low high-frequency ($`u + v \ge 6`$) energy are repeatedly selected, and the spectrum shape is obtained from the per-frequency mean. Texture energy concentrates in low frequencies, so only high frequencies are used for selection and gain.
2. Luminance is capped by a PCA noise floor: the mean of the 3 smallest eigenvalues of the 63-dimensional covariance of weak-texture blocks. Texture spreads in only a few directions while noise spreads in all 63, so this prevents texture from being mistaken for noise and removed. The finite number of samples is corrected with the Marchenko–Pastur lower bound $`\gamma = \sqrt{n / n_{\mathrm{sel}}}`$.
3. When uncertain, it is biased toward underestimating (the side that preserves detail). A clean image with only 8-bit quantization noise ($`1/12`$) left is treated as essentially 0.

**Denoising.** A two-stage structure on a sliding 8×8 DCT, BM3D without block matching. From the block mean brightness $`\bar I`$, the per-coefficient noise variance is set to $`\sigma_k^2 = S[k]\, g(\bar I)\, \lambda^2`$ ($`\lambda`$ is the strength). $`F_k`$ are the coefficients of the input block, and $`B_k`$ the coefficients of the same block of the stage-1 result image.

```math
\text{Stage 1: } \hat F_k = F_k \cdot \mathbf{1}\big[\,F_k^2 \ge \tau^2 \sigma_k^2\,\big],\qquad
\text{Stage 2: } \tilde F_k = F_k \cdot W_k,\quad W_k = \frac{B_k^2}{B_k^2 + \sigma_k^2}
```

Overlapping blocks are combined with a Kaiser window ($`\beta = 2`$) and, as in BM3D, per-block weights of $`1/(1 + n_{\mathrm{nz}})`$ in stage 1 and $`1/\sum_k W_k^2 \sigma_k^2`$ in stage 2 ($`n_{\mathrm{nz}}`$ is the number of remaining coefficients). The block grid is fixed to global coordinates, so there are no tile seams.

**JPEG quantization constraint set projection (QCS).** For JPEG, the variance of the quantization noise from the file's quantization table $`q_k`$ is added to the noise model, and the luminance DCT coefficients are brought back into the interval around the observed quantized value. The entropy-coded data is not decoded, so the observed coefficients are obtained by taking the DCT $`G_k`$ of the decoded image aligned to the file's 8×8 grid and rounding it with the quantization table.

```math
c_k = q_k \cdot \mathrm{round}\!\left(\frac{G_k - \ell_k}{q_k}\right) + \ell_k,\qquad
F_k \leftarrow \mathrm{clip}\big(F_k,\ c_k - \Delta_k,\ c_k + \Delta_k\big),\qquad
\Delta_k = \tfrac12 \rho_k q_k + \delta_k
```

$`\ell_k`$ is the level shift of 128 that exists only for DC. For AC coefficients recorded as nonzero, excluding the lowest frequencies ($`u + v \ge 2`$, real texture left by the encoder), the interval is narrowed with $`\rho_k = \rho`$ (default 0.5) to preserve texture strength, while zero coefficients, DC and the lowest frequencies (the source of blocking and ringing) use the whole interval with $`\rho_k = 1`$ to remove artifacts. $`\delta_k = 2.5\sigma_k`$ (AC only) is room to remove noise when the image was compressed with noise mixed in. (Denoising → projection) is repeated twice, and the second time the quantization noise variance is halved.

**Color.** The chroma planes are cleaned the same way, and then color boundaries smeared by chroma subsampling such as 4:2:0 are aligned with a guided filter to the restored luminance edges (locally, color $`= a \cdot`$ luminance $`+ b`$).

**Detail recovery.** For non-JPEG input, when the local energy $`v`$ of the removed amount exceeds the amount $`\tau \sigma^2`$ explained by the noise model, it is restored in proportion to the excess.

```math
\hat I \leftarrow \hat I + \mu \left(1 - \frac{\tau \sigma^2}{v}\right)(I - \hat I)\qquad (v > \tau \sigma^2)
```

### 4.10 Unsharp mask (halo suppression)

After computing the detail $`d = I - G_\sigma \ast I`$ and the local amplitude $`A = G_{2\sigma} \ast |d|`$, it is processed in the following order.

```math
d \leftarrow d \cdot \frac{d^2}{d^2 + t^2}\ \ (\text{flat-area protection}),\qquad
d \leftarrow a\, d,\qquad
d \leftarrow \frac{d}{1 + |d| / (o A + 10^{-4})}\ \ (\text{halo tolerance})
```

$`a`$ is the strength, $`t`$ the flat-area protection, and $`o`$ the halo tolerance. The last formula is a soft-knee limiter, so at large edges the overshoot cannot exceed $`o`$ times the local amplitude. A radius of 0 is set automatically according to the scale.

### 4.11 Directional smoothing (staircase removal)

The structure tensor $`J = G \ast (\nabla Y \nabla Y^{\top})`$ is computed on the original; the eigenvector of the small eigenvalue gives the tangent direction $`\mathbf t`$, and the large eigenvalue $`\lambda_1`$ gives the strength $`w = \min(1, \lambda_1 / 0.0016)`$. On the upscaled image, averaging is done only along the tangent direction.

```math
\bar U = 0.34\,U(\mathbf x) + 0.22\,\big(U(\mathbf x + \ell \mathbf t) + U(\mathbf x - \ell \mathbf t)\big) + 0.11\,\big(U(\mathbf x + 2\ell \mathbf t) + U(\mathbf x - 2\ell \mathbf t)\big),\qquad
U \leftarrow U + w\,\kappa\,(\bar U - U)
```

The staircase period equals the scale, so the step $`\ell`$ is set to 0.8 times the scale (1–8). Once is not enough, so it is repeated 1–4 times in proportion to the strength. With each repetition the wobble shrinks multiplicatively while the transition width stays the same. The direction field is found in global coordinates, so tile boundaries give the same values.

### 4.12 Contour redrawing (signed distance)

When pixels lie on the segment between the dark and bright colors $`(\mathrm{lo}, \mathrm{hi})`$ of a 5×5 window of the original, and the change happens within 1–2 pixels, it is treated as a contour. The upscaled image is converted to a position $`t \in [0, 1]`$ on the segment, and a 1-pixel-wide ramp is redrawn by the signed distance to the 0.5 isoline.

```math
t' = \mathrm{clip}\!\left(0.5 + \frac{t - 0.5}{|\nabla t|},\ 0,\ 1\right),\qquad
U \leftarrow U + \gamma\,w_{\mathrm{fit}}\big(\mathrm{lo} + t'(\mathrm{hi} - \mathrm{lo}) - U\big)
```

$`|\nabla t|`$ is the gradient per output pixel. The actual computation uses as target $`U + (t' - t)(\mathrm{hi} - \mathrm{lo})`$ clipped to $`[\mathrm{lo}, \mathrm{hi}]`$, and the blending weight $`\gamma\,w_{\mathrm{fit}}`$ is a weight built at the original resolution from the redrawing strength and the fit (how far the central 3×3 deviates from the segment), spread bilinearly. Redrawing at the 50% isoline preserves area (density) only for lines wider than 1 pixel, so thin lines whose width, measured as the integrated density across the line, is around 1 pixel are left untouched to prevent wiggling.

### 4.13 Pixel-art contour restoration

The shape is determined by the DCCI field (45° · 2:1 staircases become straight lines), and the color of each subsample is fixed to whichever of the 4 original colors of the original 2×2 cell it belongs to is closest to the field value. Each output pixel averages 4×4 subsamples, antialiasing only the contours, so new colors appear only at contour boundaries. If the 4 colors of a cell are the same, a fast path skips it.

### 4.14 White balance consensus

「Color auto」 removes saturated (0.95 or more) and nearly black (0.004 or less) pixels from the analysis image (linear light) and estimates the illuminant color with three estimators.

```math
e^{\mathrm{sog}}_c = \Big(\tfrac1n \sum f_c^{6}\Big)^{1/6},\qquad
e^{\mathrm{ge}}_c = \Big(\tfrac1n \sum |\nabla (G \ast f_c)|^{6}\Big)^{1/6},\qquad
e^{\mathrm{wp}}_c = Q_{0.99}(f_c)
```

These are shades of gray (mean), gray edge (edges) and white patch (bright areas, 99th percentile). $`G`$ is a $`[1,4,6,4,1]/16`$ smoothing. The strength $`a`$ is set from the angle $`\theta`$ each estimate makes with neutral $`(1,1,1)`$.

```math
a =
\begin{cases}
0 & \theta_{\min} < 3^\circ \ \text{or}\ \theta_{\min} > 28^\circ \ \text{or}\ \theta_{\max} - \theta_{\min} > 10^\circ \\[2pt]
\min\!\big(1,\ \theta_{\min} / \theta_{\mathrm{sog}}\big) \cdot h,\quad h = \begin{cases} \tfrac12 & \theta_{\mathrm{wp}} < 0.8\,\theta_{\mathrm{sog}} \\ 1 & \text{otherwise} \end{cases} & \text{otherwise}
\end{cases}
```

The three conditions are, in order, no cast, a scene's own color hard to attribute to lighting, and mixed lighting. If the bright areas are tinted less than the mean, the subject's color is taken to have pulled the mean, and only half is corrected. The strength is rounded to steps of 0.05. During processing, channel gains are computed from at most 150,000 pixels excluding semi-transparent (alpha under 0.5), saturated and nearly black pixels, normalized to preserve overall luminance ($`0.2126R + 0.7152G + 0.0722B`$), blended as $`1 + (g - 1)a`$, and then clipped to 0.55–1.8.

### 4.15 9-class classifier

Multinomial logistic regression is applied to $`\mathbf z`$, the 27-dimensional features $`\mathbf x`$ standardized by the training mean and standard deviation.

```math
P(k \mid \mathbf x) = \frac{\exp(\mathbf w_k^{\top} \mathbf z + b_k)}{\sum_j \exp(\mathbf w_j^{\top} \mathbf z + b_j)}
```

Since the number of samples differs by class, class-balancing weights were used, and the L2-regularized cross-entropy was minimized with L-BFGS-B. Features affected by JPEG quality (single-color blocks, noise-free blocks, noise floor, smooth blocks) are multiplied by a quality reliability $`r = \mathrm{clip}((q - 50)/40,\ 0.05,\ 1)`$. The reasons for the decision are the per-feature contributions $`z_j (w_{k_1 j} - w_{k_2 j})`$ to the logit difference between first place $`k_1`$ and second place $`k_2`$, in descending order.

### 4.16 Rendering budget

For scale $`L`$, original pixel count $`n_0`$ and result pixel count $`N`$, the peak memory of each stage is estimated as follows (bytes).

```math
\begin{aligned}
M_{\mathrm{prep}} &= \max\big(32\,n_0,\ \ 4 c\, n_0 + 32\, n_0 s_k^2\big) && \text{source preparation, staged upscaling intermediate stages} \\
M_{\mathrm{fixed}} &= M_{\mathrm{other}} + M_{\mathrm{src}}^{1} + M_{\mathrm{keep}} + n_w (T + 64 + 2t + 2p)^2 (48 + 24[t > 0] + 32[p > 0]) + 1.5\,W \max(4T, 1024) + 1.1\,\rho\, c\, b\, N && \text{other originals · prepared originals, remaining planes, worker tiles, save bands, compressed PNG} \\
\text{guaranteed} &\iff M_{\mathrm{other}} + M_{\mathrm{src}}^{2} + M_{\mathrm{prep}} \le 0.8B \ \wedge\ 0.8B - M_{\mathrm{fixed}} \ge 4 \min\big(N, \max(4 \times 10^6, 2WT)\big)
\end{aligned}
```

$`c`$ is the number of channels, $`s_k`$ the intermediate scale of staged upscaling, $`n_w`$ the number of workers, $`T`$ the tile size, $`t`$ the result margin of texture revival (0 when off), $`W`$ the result width, and $`B`$ the memory budget. $`M_{\mathrm{src}}^{2}`$ is all the other prepared originals left in the cache (both remain while a new one is being prepared), and $`M_{\mathrm{src}}^{1}`$ is the most recent one of them (only one remains once the new one is inserted). $`b`$ is bytes per channel (1 for 8 bits, 2 for 16 bits), and $`\rho`$ is the maximum compression ratio of the saved PNG. It was measured with the app's encoder by upscaling the 47 images of `data/big` at 1.1 · 1.25 · 1.5 · 2 · 3 · 4×. At 2× and above it is the larger of the case using only presets and the case also using 「Auto」's large-scale restoration search values (`AUTO_BIG`); below 2× search values are not used, so presets only (`research/budget/png_ratio.js`, `research/results/png_ratio.json`). The larger value of the two measured scales bracketing the result scale is used (the 4× value for 4× and above). An area scale of 1.25× is also 1.12× in width and height, so it is within the measured range, and below 1.1× it is counted at uncompressed size. $`p`$ is the result margin of 「Perceptual」 (the two blurs and the neighborhood minimum/maximum; 0 for 「Accuracy」). The compression ratio was also measured with 「Perceptual」 (strength up to 150%, texture off and on), and the larger value of the two goals is used (`research/results/png_ratio_pop.json`).

| Scale | 8-bit, texture off | 8-bit, texture on | 16-bit, texture off | 16-bit, texture on |
|---|---|---|---|---|
| 1.1× | 0.721 | 0.767 | 0.928 | 0.933 |
| 1.25× | 0.686 | 0.766 | 0.913 | 0.928 |
| 1.5× | 0.650 | 0.760 | 0.898 | 0.926 |
| 2× | 0.688 | 0.790 | 0.876 | 0.933 |
| 3× | 0.615 | 0.751 | 0.842 | 0.931 |
| 4× and above | 0.562 | 0.731 | 0.817 | 0.934 |

8-bit values are with dither on, and texture on is the larger of 25% and 100%. The last term is the cache of two tile rows (at least 4 megapixels) needed for saving. The memory budget $`B`$ is 15% of `navigator.deviceMemory` (400–1500 MB); if unavailable, 50% of the JS heap limit (at most 1500 MB); and if that is also unavailable, 600 MB. On mobile it is 12% (at most 400 MB, 180 MB if unavailable).

### 4.17 8-bit dither

Interleaved gradient noise (IGN, Jimenez 2014), determined only by the result coordinates $`(X, Y)`$, is added to the continuous sRGB value $`s = 255\,\mathrm{srgb}(v)`$ just before rounding.

```math
\nu(X, Y) = \mathrm{frac}\!\big(52.9829189 \cdot \mathrm{frac}(0.06711056\,X + 0.00583715\,Y)\big) - \tfrac12,\qquad
b = \big\lfloor s + \tfrac12 + 0.98\,\nu \big\rfloor
```

The amplitude is ±0.49 levels, so values with $`|s - k| < 0.01`$ (an exact code value $`k`$) stay $`k`$. The continuous sRGB value is obtained by interpolating the 16-bit encoder's $`\sqrt v`$-axis table, with interpolation error of at most 0.002 levels. The same $`\nu`$ is used for the three channels, and alpha is not dithered.

### 4.18 Texture revival

From the gamma-approximated brightness $`Y = \sqrt{\ell}`$, $`\ell = 0.2126R + 0.7152G + 0.0722B`$ of the original (linear light, premultiplied), the local strength of the second-finest band and the edge mask are measured.

```math
a = \mathrm{erode}_{5\times5}\!\left(\sqrt{G_{1.5} \ast (G_1 \ast Y - G_2 \ast Y)^2}\right),\qquad
m = \big(1 - c^2\big)^2,\quad c = \frac{\sqrt{(J_{xx} - J_{yy})^2 + 4J_{xy}^2}}{J_{xx} + J_{yy}}
```

$`G_\sigma`$ is a Gaussian, $`J = G_{1.5} \ast (\nabla Y \nabla Y^{\top})`$ is the structure tensor, and $`c`$ is the coherence. The 5×5 minimum (erosion) removes the strength of narrow strong structures such as edges, thin lines and corners, and keeps only widely spread texture. $`m`$ approaches 0 at edges running in one direction.

At the result resolution (scale $`L`$), unit-variance noise keeping only the band above the original's Nyquist is created. $`h(X,Y,s)`$ is an integer hash mapped to [0, 1).

```math
n = \sqrt3\Big(\textstyle\sum_{s=1}^{4} h(X, Y, s) - 2\Big),\qquad
\eta = \frac{n - G_{0.375L} \ast n}{\sqrt{1 - 2k_0^2 + (\sum_i k_i^2)^2}}
```

$`k_i`$ is the 1-D Gaussian kernel, and the denominator makes the variance of this difference exactly 1. $`\sigma = 0.375L`$ is the value whose response halves at the original's Nyquist. From the target variance and the variance of the same band already present in the result, only the missing share is added.

```math
T = \big(\alpha\, C\, \sqrt{\log_2 L}\; a\, m\big)^2,\qquad
E = G_{0.5L} \ast \big(Y_U - G_{0.375L} \ast Y_U\big)^2,\qquad
\Delta = \mathrm{clip}\!\big(\sqrt{\max(0, T - E)}\;\eta,\ \pm 0.4\,Y_U\big)
```

$`\alpha`$ is the strength slider, $`C = 6.256`$ is a constant fitted by measurement ([9.8](#98-texture-revival)), and $`Y_U`$ is the gamma-approximated brightness of the result. Finally, the three channels (premultiplied) are multiplied by the same ratio $`f = \big((Y_U + \Delta)/Y_U\big)^2`$, keeping hue and alpha unchanged. When brightening ($`f > 1`$), it is reduced to $`f \le \max\!\big(1,\ \min(1, A) / \max(R, G, B)\big)`$ so that the largest channel does not exceed its upper bound (alpha, or 1 if opaque). Otherwise, in bright, highly saturated colors only one channel gets clipped and the hue changes (e.g. 250·127·73 became 255·130·76; fixed in v1.1.1). The noise, the original statistics and the result energy window are all determined in global coordinates and correspondingly larger margins are computed, so the result is the same regardless of tiles.

### 4.19 Gamut detection and the Display P3 profile

The primaries matrix $`(\mathbf r, \mathbf g, \mathbf b)`$ (XYZ relative to D50) of an ICC profile is compared with sRGB's values, and if even one component differs by more than 0.02, it is treated as wide gamut. LUT profiles without a matrix are judged by their description string (P3, Adobe, 2020, ProPhoto). PNG cICP and AVIF nclx are treated as wide if the primaries code is not 1 (BT.709 = sRGB) or 2 (unspecified). For AVIF · HEIF, `pitm` (primary item) of the `meta` box and `ipma` (item ↔ property index) · `ipco` (properties) inside `iprp` are read, and only the `colr` associated with the primary image is considered. Even if a thumbnail's or auxiliary image's `colr` comes first, the primary image is what counts. If this structure cannot be read, the first `colr` found is used.

The Display P3 profile attached when saving is generated by the app itself (ICC v4.3, 532 bytes). The primaries are obtained by deriving the RGB→XYZ matrix from the P3 chromaticity coordinates (0.680, 0.320), (0.265, 0.690), (0.150, 0.060) and the D65 white, then adapting to D50 with the Bradford transform.

```math
\mathbf r = (0.515119,\ 0.241189,\ -0.001050),\quad
\mathbf g = (0.291978,\ 0.692244,\ 0.041879),\quad
\mathbf b = (0.157103,\ 0.066567,\ 0.784071)
```

The curve is the same parametric curve as sRGB (type 3: $`\gamma = 2.4`$, $`a = 1/1.055`$, $`b = 0.055/1.055`$, $`c = 1/12.92`$, $`d = 0.04045`$). These values match the primaries of Apple's Display P3 profile within $`10^{-5}`$.

### 4.20 Metrics

```math
\mathrm{PSNR} = 10 \log_{10} \frac{255^2}{\frac{1}{3n}\sum (U - I_{\mathrm{gt}})^2},\qquad
E_{\mathrm{dir}} = \frac{\sum_p m_p\, \arccos\!\big(|\hat{\mathbf g}^{U}_p \cdot \hat{\mathbf g}^{\mathrm{gt}}_p|\big)}{\sum_p m_p}\quad (m_p \ge 80)
```

$`E_{\mathrm{dir}}`$ is the edge direction error. At strong edges of the original (Sobel magnitude 80 or more, a step of about 20 levels), the angle by which the gradient directions of the result and the original disagree is averaged, weighted by edge strength. Staircases push gradient directions toward the axes and increase it. SSIM uses the definition of Wang et al. (2004).

### 4.21 Perceived sharpness

In 「Perceptual」 mode, two bands are added to the result luminance $`Y`$ ($`\sqrt{\ell}`$ in linear light, $`\ell`$ for gamma values; $`\ell`$ is the luminance of the channels clipped to 0–cap). $`L`$ is the scale, $`G_\sigma`$ a Gaussian (three-box approximation), and $`a`$ the perceptual strength.

```math
d_1 = Y - G_{0.5L}Y,\qquad d_2 = G_{0.5L}Y - G_{2L}Y,\qquad
\tilde d = d\,\frac{d^2}{d^2+\tau^2}\quad(\tau = 0.004)
```

```math
\Delta = a\Big(k_1\,\tilde d_1 + k_2\,\frac{\tilde d_2}{1+|\tilde d_2|/c_2}\Big),\qquad
Y' = Y + \Delta\,\frac{h}{h+|\Delta|},\qquad
h = \begin{cases}\max_{\mathcal N}Y - Y & \Delta>0\\ Y-\min_{\mathcal N}Y & \Delta\le 0\end{cases}
```

$`d_1`$ is the edge slope near the source Nyquist frequency, $`d_2`$ the local contrast at the scale of 1–4 source pixels. $`\tilde d`$ is a threshold of the same form as the unsharp mask, so small amplitudes such as noise and JPEG traces are hardly raised. Local contrast is raised less as its amplitude grows ($`c_2 = 0.05`$), leaving already clear contrast as it is and bringing out buried contrast. $`\mathcal N`$ is a square neighborhood of radius $`\mathrm{round}(0.75L)`$ (inside the image only), and $`h`$ the room left to the neighborhood minimum or maximum. $`\Delta\,h/(h+|\Delta|)`$ only approaches that room smoothly and never crosses it, so no new extrema (halos) appear, and since values do not stick to the bound, surfaces do not clump (posterize). A first version clipped exactly to the neighborhood range; surfaces clumped and looked like an oil painting, so it was changed. The new luminance is applied as a ratio $`s = Y'/Y`$ ($`s^2`$ in linear light) multiplied equally into the three premultiplied color channels, so hue, gray and alpha are kept; when brightening, the ratio is reduced so the largest channel stays under the cap (the same handling as texture revival). With alpha, luminance is measured on the un-premultiplied color, the blurs are opacity-weighted averages, and nearly transparent pixels (alpha below 2%) are left out of the neighborhood minimum/maximum, so a boundary where only opacity changes while the color stays the same gets no fringe.

$`k_1 = 0.5`$ and $`k_2 = 1.5`$ were chosen on the exploration images. The 16 combinations $`k_1 \in \{0.5, 1, 2, 3\}`$ × $`k_2 \in \{0, 0.5, 1, 1.5\}`$ were measured on 22 images × 3 conditions (4× clean, 4× degraded, 8× degraded; 66 cases), comparing the median log error $`|\ln A| + |\ln C|`$ of edge slope $`A`$ and local contrast $`C`$ against the ground truth. $`A`$ is the ratio of slopes measured at ground-truth edges (top 10% of slope) after blurring both with $`\sigma = 0.5L`$; $`C`$ is the RMS ratio of the $`G_{0.5L}-G_{2L}`$ band. All 16 had a smaller error than 「Accuracy」 (0.418), from 0.343 to 0.410; among them the one adding the least halo and flat-area noise was chosen (error 0.357, halo +0.20 levels, flat-area noise ×1.14, SSIM +0.004). The results are in `research/results/pop_grid.json`.

---

## 5. The physical world

| Physical phenomenon | Model in Upscaly |
|---|---|
| A sensor pixel sums all the light falling on its area | Downscaling model = linear-light area average $`D`$. The consistency correction target $`DU = I`$, the staged-upscaling measurement and the downscaling of the search data all use this model |
| Blur from lens and scanner optics | Gaussian point spread function (σ 0.45–0.95). Inverted with RL deconvolution |
| Sensor noise strength depends on brightness | Noise spectrum with a gain per brightness bin $`g(I)`$ |
| JPEG rounds DCT coefficients to integers | A quantization interval of width $`q_k`$ per coefficient. The true original lies somewhere within that interval |
| Chroma subsampling (4:2:0) records color at half resolution | Guided filter aligning color boundaries to luminance edges |
| Illuminant color tints even the bright areas | Three-estimator consensus rule |
| Semi-transparent edges mix foreground and background light by area ratio | Premultiplied alpha. The same operations on color and alpha |
| The eye perceives brightness nearly logarithmically | Saving in sRGB; direction and edge decisions on a gamma-luminance guide plane |
| The power spectrum of natural images is roughly $`1/f^2`$ (similar energy per octave) | Texture revival's estimate of the missing octave: original band strength × $`\sqrt{\log_2 L}`$ |
| Film grain is fine particles at random positions | Texture revival is grain matched only to the amount of texture. The position and shape of the texture are not in the original |
| The eye is sensitive to steps (bands) over wide areas and insensitive to very fine noise | 8-bit dither: moves low-frequency quantization error into high-frequency noise |
| Each display and camera has a different range of reproducible colors (gamut) | Reads the original color profile, processes with a P3 canvas · P3 profile, and saves |

---

## 6. System

### Files and runtime environment

The whole tool is a single file, `upscaler.html` (about 6,140 lines, 360 KB), containing HTML, CSS and JavaScript. There are no external libraries and no server is needed. Only the screen font (IBM Plex Sans KR) is fetched from Google Fonts, and if it cannot be fetched, the system font is used. It uses `CompressionStream` for streaming PNG saving and Web Workers for parallel computation, and falls back to slower paths if either is missing. If the device is in dark mode, it uses a dark theme.

### Components

| Layer | Main functions | Role |
|---|---|---|
| Color space | `S2L`, `L2S`, `enc16Linear` | sRGB ↔ linear-light tables, 16-bit encoder |
| Engine | `ENGINE_FACTORY`, `PORT` | Pure computation shared by main thread and workers |
| Interpolation | `dcciStageMulti`, `ediUpsample`, `resample2D`, `interpRegion` | DCCI, kernel resampling |
| Constraints | `cellRange`, `clampRegion`, `consistencyCorrectPlanar` | Cell range, consistency (Thomas) |
| Restoration | `estimateNoise`, `sdctDenoise`, `qcsLuma`, `guidedFilter`, `restoreTile`, `richardsonLucy`, `denoise`, `deblock` | Noise · JPEG · blur |
| Contours | `edgeField`, `antialiasRegion`, `crispFields`, `steepenRegion`, `pixelTile` | Directional smoothing, redrawing, pixel art |
| Tiles | `renderTile`, `tileJob`, `cropRectFor`, `outMarginFor` | Computing one tile, margin planning |
| Source preparation | `prepareSource`, `dctRestorePool` | Original-resolution corrections, staged upscaling intermediate stages |
| Parallelism | `initPool`, `poolRun`, `runLimited` | Worker pool, queue, limit on concurrent pieces |
| Cache | `tcGet`, `tcPut`, `tcMax`, `srcGet` | Result tile LRU, 2 prepared originals |
| Auto | `analyzeImage`, `autoClassify`, `autoPick`, `AUTO_BIG`, `autoAA`, `autoStage`, `pickAutoScale`, `renderBudget` | Analysis, classification, settings, scale |
| Color processing | `estimateWB`, `colorAuto` | White balance |
| 8-bit dither | `ditherTile`, `srgbF` | Coordinate-determined IGN dither |
| Texture | `texFields`, `texNoise`, `addTexture`, `texMargin` | Original texture strength, band noise, adding only the missing share |
| Gamut | `P3_OK`, `sniffWideGamut`, `iccIsWide`, `displayP3Icc`, `p3IccpChunk` | Wide-gamut detection, P3 canvas, P3 profile |
| Saving | `createPngEncoder`, `exportPng`, `exportViaCanvas`, `saveAll` | PNG per tile row, batch saving |
| Screen | `render`, `drawTiles`, `detectSubjects` | Comparison view, subject box |

### Data flow and cache

Each image separately holds its source canvas, JPEG grid and quality, noise model (computed once when first needed), subject box and settings. A result carries a snapshot of the settings at the time it was made, so even if settings change later, saving and recomputation produce the same values as that result.

Result tiles are kept in an LRU cache. The cache limit is 60% of the memory budget, and if creating a canvas actually fails, 80% of the cache size at that moment is learned as this device's limit. At a scale chosen by auto scale, the limit is lowered further according to the guarantee conditions. For results that do not fit entirely in the cache, only half the cache is computed first, starting from the center, and the rest is computed when it becomes visible on screen or when saving. Tiles visible on screen but not yet available are filled with an enlargement of the original and their computation is queued in the background.

### Determinism: screen = saved result

The screen, saving and recomputation all use the same `renderTile`. Operations that look at neighbors (interpolation, consistency, unsharp, directional smoothing, redrawing) compute an extra margin of the needed radius and then crop it, and the DCT block grid and direction field are fixed to global coordinates. So the result is the same regardless of tile size, order and number of workers. Even when computed finely in 64px tiles, tile seams are at most 1 level, and the difference between two computations with the same settings is 0 ([9.1](#91-invariants)).

---

## 7. Issues and fixes

### Image quality

| Issue | Cause | Fix |
|---|---|---|
| White rims around edges (ringing · halos) | The kernel's negative tails create extremes not in the original | DCCI 4-neighbor clip, cell range limit, unsharp knee limiter |
| Shrinking the result does not give the original | Even if interpolation passes through the original samples, the area averages do not match | Solve consistency correction exactly at the original resolution (Thomas) |
| Consistency and cell range break each other | Satisfying one breaks the other | Alternating projection, twice |
| Color bleeding at semi-transparent edges (compositing error up to 29 levels) | Alpha and color went through different operations | Make alpha go through the same operations as color (0.1–4 levels) |
| Staged upscaling actually gets worse (−0.4 to −0.8 dB) | The cell range · consistency reference was the intermediate image | Use the original as reference (+0.03 to 0.06 dB) |
| The original's staircase comes back in line art | Consistency treats the original's staircase as the correct answer | Redrawing and directional smoothing for line art · logos; whether to use consistency is decided per kind in the search |
| Thin lines wiggle | 50% isoline redrawing cannot preserve the density of lines thinner than 1 pixel | Measure line width by integrated density and leave thin lines as they are |
| DCCI misjudges direction in noise | Noise shakes the directional variation | Add a noise floor $`n_{\mathrm{off}}`$ to the variation (Immerkær σ) |
| Texture mistaken for noise and removed | Texture blocks inflate the noise spectrum | Cap with the PCA noise floor, select by high frequencies only |
| JPEG restoration erases even the detail in the file | Denoising alone does not know the original coefficients | Quantization constraint set projection (QCS) |

### Color

| Issue | Cause | Fix |
|---|---|---|
| Presets change colors too | Presets used to include white balance | Color processing split out as a separate feature, default 0, only the 「Color auto」 button |
| Orange cats · sunsets mistaken for a cast | Only a single mean estimator was used | Three-estimator consensus (strong color bias over 54 pairs 13.9° → 4.0°) |
| Dark boundaries clump | Interpolation in gamma space | Compute in linear light |
| Phone photos turn dull after upscaling | The sRGB canvas squeezed Display P3 colors into sRGB (P3 red as P3 values 234,51,35) | Read the original profile, process with a P3 canvas and save with a P3 profile (cICP · iCCP) (v1.1.0) |
| Stair-like bands in heavily enlarged skies | 8-bit rounding bands widen by the scale | Coordinate-determined dither (v1.1.0) |
| Adding texture gives white dots at corners · dark areas | Band strength of narrow strong structures is large, and in dark areas the same change is relatively large | 5×5 erosion on texture strength, limit a pixel's change to ±40% of brightness (√) |
| Turning off 「Compute in linear light」 makes texture overshoot 1.8× | Gamma values were treated as linear light and √ and squaring were applied once more | In gamma mode the brightness axis is the gamma value as is, and the ratio is not squared. R on 12 photos: 2.24 before the fix, 1.32 after (linear-light mode 1.22, v1.1.1) |
| Texture changes the hue of bright, highly saturated colors | Even multiplying the three channels by the same ratio, only the largest channel gets clipped at its bound | Limit the brightening ratio to the headroom of the largest channel (alpha · 1). Bright red at 4×: maximum change in linear color ratio 14.0% → 2.9% (8-bit quantization level, v1.1.1) |
| Texture strength overshoots on clean input and falls short on degraded input | The finest band is eroded first by blur · JPEG | Measure on the band one octave lower, subtract the high frequencies already in the result, and add only the missing share |

### Stability and I/O

| Issue | Cause | Fix |
|---|---|---|
| The app freezes right at startup | `updateSizeNote`, called at startup, read a `let` global declared later, causing a TDZ error | Declare such globals with `var` or place them earlier |
| A late-finishing measurement is applied to a new image | Race between asynchronous measurements | Auto request number `AUTO_SEQ` and `alive()` check |
| A large canvas is blank without any exception | The browser's hidden area limit | If the source canvas is entirely transparent, read the original directly in 1024² pieces to see whether any pixel is visible; for save assembly, find one visible pixel in the result, check that spot on the canvas, then learn the limit. Even an image with a single visible dot is detected (v1.1.1. Previously only the bottom-right pixel was checked, so it was missed if that pixel was transparent to begin with) |
| AVIF wide gamut detected as sRGB | The first colr box in the file was used, which may belong to a thumbnail | Only the colr associated with the primary image via `pitm` · `ipma` · `ipco` (v1.1.1) |
| PNG wide gamut detected as sRGB | It stopped at an sRGB · iCCP chunk placed before cICP | Look at all color chunks before IDAT, then cICP > iCCP > sRGB. If the cICP primaries are unspecified (2) or a reserved code, use iCCP (v1.1.1) |
| 「Upscale in stages」 changed right after opening an image reverts | The measurement deciding the checkbox's initial value finished late and overwrote it | Increment a number (`STAGE_SEQ`) whenever the user or 「Auto」 decides, and do not apply if it differs from when the measurement started or conditions changed (v1.1.1) |
| Edge color bleeding · tile seams in transparent images with staged upscaling | Only alpha went through ordinary resampling in the intermediate stages | Alpha goes through the same operations as color (DCCI 4× edge color max 19–38 → 1–4 levels, logo seam 206 → 1, v1.1.1) |
| Decoding a large file never finishes | Cases where neither `onload` nor `onerror` arrives | Try three paths in turn with size-proportional time limits |
| Environments where workers are blocked | Security policy | Check for a response within 5 seconds, then fall back to the main thread |
| Memory exceeded because the result is large | A canvas for the whole result | Tile cache and streaming PNG |
| Settings go out of sync if the scale is changed after Auto | Auto decides based on that scale | Notify via `AUTO_LAST` and ask to press again |
| Clean-input validation processed like JPEG input | The validation script passed JPEG quality 85 even for clean input | Revalidated without JPEG information; logo · comic search values only for JPEG input (v1.1.0) |

---

## 8. Optimization

Speed optimizations were added only when the output is byte-for-byte identical. The 16-bit output and statistics of 134 setting combinations are compared against the baseline (`tests/engine_regression.js`). The exception is the DCCI internal grid table, the first row of the table below. It is a trade-off adopted after measuring that it becomes faster at the cost of a slight change in output (±0.007 dB), and the current baseline is the version after this table was introduced.

| Optimization | Details | Effect |
|---|---|---|
| DCCI internal grid table | 4× grid instead of 8× grid at 6 · 8 · 12×. Within ±0.007 dB over 25 presets × 1,800 pairs | 25–30% faster |
| Shared DCCI direction weights | Computed once on the guide plane and used for all channels | |
| Cell range cache | Per-cell minimum and maximum only once per channel plane | |
| Grouped consistency channels | Footprints, coefficients and interpolation weights shared across channel planes without copying them into an interleaved buffer | |
| Total | 1024×768 → 4× 「Standard」(표준) | 13.9 s → 12.0 s, 134 combinations byte-identical |

In addition, Gaussians are approximated by three box blurs (Kutskir's method), the bilateral filter's brightness weights use a table, and 16-bit bands are recomputed in 128 rows to save memory. A 4× grid was also tested at 2× and below, but although the mean rose by +0.013 dB, cases that got worse (24%) outnumbered cases that got better (21%) and the time doubled, so it was not adopted.

The remaining bottlenecks are DCCI interpolation (per channel), consistency correction and resampling. It is hard to reduce them substantially without changing the output.

---

## 9. Validation

### 9.1 Invariants

`tests/verify_suite.js` loads the app script as is onto a fake DOM and runs the actual processing path to the end.

| Item | Expected | v1.1.1 |
|---|---|---|
| Tile seams (29 presets, 64px tiles) | At most 1 | All at most 1 |
| Staged upscaling tile seams (lanczos3 4× · edi 2× · pixelart 4×) | 0 | 0 · 0 · 0 |
| Determinism (5 presets computed twice) | 0 | 0 |
| 1× no-op identity | 0 | 0 |
| Nearest 3× bit exactness | 0 mismatched values | 0 |
| Grayscale preserved (max R · G · B difference over 29 presets) | 0 | 0 |
| Alpha edge color (alpha 5% or more) | At most 4 | 4 |
| Staged upscaling 4× alpha edge color · tile seams (photo · logo × lanczos3 · edi) | At most 4 · at most 1 | 4 · 0, 0, 1, 1 (v1.1.0: color 22–38, logo seam 206) |
| Linear-light table round trip (256 levels) | 0 mismatches | 0 |

The invariants are measured with 8-bit dithering on (default). Apart from the staged-upscaling alpha item, they are the same as v1.0.0. Other tests:

| Test | Contents | v1.2.2 |
|---|---|---|
| `e2e_presets.js` | Processing 8 presets + 8 · 16-bit saving | ✓ 8, 45,711 · 239,731 bytes (8-bit is +5% from v1.0.0's 43,446 due to dither) |
| `engine_regression.js` | 16-bit output · statistics of 134 combinations compared against the baseline | 134/134 byte-identical |
| `worker_test.js` | Worker tile = main computation | Byte-identical |
| `alpha_probe.js` | Semi-transparent edge color error per preset | |
| `e2e_autoscale.js` | Auto scale: with a 600 MB budget, 512² · 800×600 → 8×, 4000×3000 → 2×. Guaranteed even after pressing 「Auto」 with nearest · DCCI three stages (texture on), and larger scales are not guaranteed with that scale's 「Auto」 settings (6 cases). 4000×3000 with classification fixed in the UI, 「Auto」 at 1.25× → 2× | 4000×3000 was 4× in v1.1.0. At 4× the result is 192 MP and the compressed PNG alone piles up to about 300 MB, so it is not guaranteed. In the UI case, versions before v1.1.1 stayed at 1.25× with 「No guarantee」(보장 없음) |
| `e2e_auto_stale.js` | Notice on scale change after Auto | |
| `e2e_stage_force.js` | Forcing staged upscaling on · indicator, keeping user changes during the initial measurement | ✓ 2 (v1.1.0 overwrote both) |
| `blank_canvas.js` | Blank canvas detection: 6 source reads, 4 canvas saves | ✓ 10 (v1.1.0 missed 4 cases where the bottom-right was transparent · only one dot was visible) |
| `gamut_sniff.js` | Wide-gamut detection: 5 AVIF primary-image colr cases, 4 PNG color chunk orders, 5 cases with over 2 MB of metadata · boxes before the color information (JPEG extended XMP, PNG text chunks, AVIF mdat before meta), 4 cICP primaries codes (unspecified · reserved · sRGB) | ✓ 18 (v1.1.0 got 10 wrong) |
| `dither_probe.js` | 8-bit dither banding metric | [9.7](#97-8-bit-dithering) |
| `texture_seam.js` | Texture on: seams · determinism · grayscale · hue of bright colors | Seams at most 1 (same as texture off), two computations 0, grayscale 0, bright red color ratio change 2.9% (v1.1.0: 14.0%) |
| `pop_seam.js` | 「Perceptual」: tile seams · determinism · grayscale · no halo (within the neighborhood range of the Accuracy result) · gamma mode · hue of bright colors · opacity-only boundaries | Seam at most 1 (same as Accuracy), 0 between two runs, grayscale 0, at most 0.9 levels beyond the neighborhood range (8-bit rounding), bright red color ratio change 2.8%, color difference at opacity-only boundaries at most 1 (24 before the fix, 18 on a white ramp) |
| `p3_browser.js` | P3 color preservation in real Chromium (primaries · mixed colors, including simulated environments where blob: images are blocked and `createImageBitmap` converts to sRGB) | [9.9](#99-display-p3) |

### 9.2 Data

`data/ds` holds 567 originals divided into 9 kinds. Variants made from the same originals by JPEG quality, downscaling, blur, noise, simulated scanning and so on are included alongside.

| Kind | Photo | Illustration | Painting | Comic | Line art | Document | UI | Logo | Pixel | Total |
|---|---|---|---|---|---|---|---|---|---|---|
| Count | 135 | 152 | 36 | 45 | 40 | 55 | 38 | 43 | 23 | 567 |

`data/big` is for the large-scale restoration search (`research/search/prep_big.py`). For each kind, 3 search images and 3 validation images are set apart, and the 256px patch with the most detail in each original is taken as ground truth to build four inputs.

| Condition | Input | Ground truth |
|---|---|---|
| Realistic degradation 2 · 4 · 8× | Linear-light area downscale → blur σ0.6 → JPEG 85 (4:2:0) | 256px patch |
| Clean input 4× | Linear-light area downscale only | 256px patch |
| Large image 4× (validation only) | Realistic degradation of the central 384–512px | That patch |

### 9.3 Graphics presets and graphics auto

The 11 presets cel, digital painting, poster, comic, line art, document, handwriting, drawing, UI, screen text and logo have consistency on, with directional smoothing lowered to 50% for cel and 75% for digital painting · poster. These are settings where, on 29 tuning and 30 validation images degraded with blur + JPEG, PSNR · SSIM rose by 0.2–0.6 dB and staircases on edges such as 45° (direction error) stayed within 10% of the original. When graphics auto measured and chose directional smoothing per image, the 59 graphics images gained +0.44 dB on average, with edge direction error −8.4%.

### 9.4 Large-scale restoration search

**Space.** Kernel 4 × consistency 2 × cell range 2 × sharpening amount 19 × (radius · threshold · overshoot · directional smoothing · redrawing · deconvolution multiplier · noise multiplier · JPEG restoration multiplier · staged upscaling) 10 each × deconvolution σ multiplier 5 = **1,520,000,000,000 combinations**.

**Constraints.** SSIM drop within 0.003 and edge direction error within +10%, relative to the current Auto.

**Procedure.** 1,500 random → re-evaluation of the top 150 → coordinate descent from 4 starting points (all stages, until convergence) → values stuck at the slider ends are retried with a widened range. About 55,000 evaluations in total. The log (`data/big/search.jsonl`) accumulates, so runs continue where they left off.

**Validation.** PSNR difference (dB) relative to the previous Auto on the 3 validation images set apart. It is adopted only for kinds · scales where the mean rises and at least 2 of the 3 images improve. The whole table was re-measured in v1.1.0 (`research/results/val_v3.json`). The earlier validation differed from actual use in two ways. First, JPEG quality 85 was passed even for clean input, so JPEG restoration was applied to the previous Auto used as the comparison baseline. Second, only the quality number was passed for JPEG input, not the quantization tables · grid, so the quantization interval restoration used on real JPEG files was missing. Even after fixing both, the adoption range did not change except for the JPEG condition on logos · comics.

| Kind | 2× | 4× | 8× | 4× clean input | 4× large image | Adopted |
|---|---|---|---|---|---|---|
| Photo | +1.73 | +0.36 | +0.19 | +0.94 | +0.89 | 2× and above |
| Line art | +1.22 | +1.08 | +0.32 | +0.42 | +1.67 | 2× and above |
| UI | +0.84 | +0.36 | +0.15 | +0.37 | +0.22 | 2× and above |
| Comic | +2.27 | +0.62 | +0.07 | −0.15 | +0.44 | 2× and above, JPEG input only |
| Painting | +2.37 | +1.46 | +0.25 | +1.50 | +1.51 | 2× and above |
| Document | −0.48 | +0.55 | +0.22 | +0.58 | +0.55 | 4× and above only |
| Illustration | +0.28 | +0.26 | +0.04 | −0.12 | +0.16 | Not used |
| Logo | +2.09 | +3.04 | +1.57 | +0.18 (1/3 images) | +1.36 | 2× and above, JPEG input only |

For logos, re-measuring clean input still improved only 1 of 3 images, and the edge direction error of +14.8% exceeded the constraint (+10%). Inputs that went through JPEG (2 · 4 · 8× realistic degradation, 4× large image) all passed, so the values are used only for JPEG files. For comics, the mean on re-measured clean input fell to −0.15 dB, so the same condition was attached. The clean-input numbers for photos · paintings decreased but still pass.

### 9.5 Classifier

567 images were trained on with the 27 features (`features2.py`) in `research/classifier`. Variants from the same original were kept in one group and cross-validated by group, so different versions of the same image never appear in training and testing at the same time. The grouped cross-validation accuracy is **92.9%**. The weights are in `AUTO_MODEL` in `upscaler.html` as inline JSON, and the JS decisions match Python on 567/567.

### 9.6 Color processing

With the three-estimator consensus rule, strong color bias (angle from neutral) on 54 test pairs dropped from 13.9° to 4.0°.

### 9.7 8-bit dithering

The error of the 8-bit output is measured against the 16-bit output of the same processing (`tests/dither_probe.js`). The banding metric is the RMS after blurring the error with a Gaussian σ2 (result pixels). Bands are low-frequency errors spanning wide areas, so they survive blurring, while the fine grain of dither disappears when blurred. 20 cases: a synthetic sky gradient and 9 photo · painting · illustration patches from the large-scale restoration validation set, enlarged 4 · 8×.

| Metric (mean of 20 cases, levels) | Rounding | Dither | Change |
|---|---|---|---|
| Banding metric σ2 | 0.059 | 0.033 | −44% (about −50% at 8×) |
| Banding metric σ4 | 0.040 | 0.017 | −58% |
| Overall RMS | 0.245 | 0.340 | +39% |

The overall RMS rises by as much fine noise as the dither adds. Even with dither on, the nearest 3× output has 0 mismatched values, and the maximum R · G · B difference for grayscale input is 0.

### 9.8 Texture revival

On the large-scale restoration data (256px ground truth → 4 · 8× downscale → upscale again), it is measured by the energy of the band above the original's Nyquist (Gaussian σ = 0.375 · scale difference) (`research/texture/tex_probe.js`, results `research/results/texture*.json`).

- **High-frequency energy ratio R** = result / ground truth. Upscaling cannot create this band, so R is much smaller than 1.
- **Texture map error** = RMS of the log ratio of high-frequency energy in 16×16 blocks. It checks whether texture is where it should be.
- **PSNR · SSIM.** Invented texture does not match the ground truth in position, so it is normal for these to go down.

**Fitting the constant.** On 9 cases from the 3 photo search images × (4× clean input, 4× · 8× realistic degradation), the constant giving R = 1 was found by the secant method, and the median of the 8 cases where R could be matched, $`C = 6.256`$, was set as 100% (5.915 in v1.1.0; it grew slightly once brightening was reduced by the channel headroom). The settings are the presets · search values chosen by classifying the downscaled input, as 「Auto」 does. It spreads widely from 2.4 to 12 per image, because the amount of texture differs by photo and a single original band cannot estimate the missing energy precisely. Checked against the ground-truth images, the original's second band × number of octaves was the most accurate of the four prediction formulas tested on photos (energy ×÷1.45).

**Validation (3 photo validation images × 4 conditions = 12 cases).**

| Strength | R | Texture map error | Cases where both improved | PSNR | SSIM |
|---|---|---|---|---|---|
| Off | 0.11 | 1.04 | | | |
| 25% | 0.27 | 0.70 | 12/12 | −0.39 dB | −0.05 |
| 100% | 1.22 | 0.57 | 12/12 | −2.24 dB | −0.18 |

**Other kinds.** Values measured with the same constant (`research/results/texture.log`, `texture_25.log`). The settings are the same as 「Auto」: the downscaled input is classified to choose a preset, and the large-scale restoration search values are read as is from the app's `AUTO_BIG` and used only under their kind · scale · JPEG conditions (the v1.1.0 measurement chose presets by ground-truth image · data classification and also used search values for illustrations with JPEG input, which differed from the actual app).

| Kind (cases) | Strength | R | Texture map error | Cases where both improved | PSNR |
|---|---|---|---|---|---|
| Painting (12) | 25% | 0.18 → 0.78 | 0.88 → 0.41 | 12/12 | −0.63 dB |
| Painting (12) | 100% | 0.18 → 8.51 | 0.88 → 0.94 | 7/12 | −4.50 dB |
| Illustration (12) | 25% | 0.07 → 0.11 | 1.12 → 1.01 | 10/12 | −0.12 dB |
| Illustration (12) | 100% | 0.07 → 0.41 | 1.12 → 0.99 | 6/12 | −1.02 dB |
| Flat graphics (24) | 25% | 0.27 → 0.28 | 0.67 → 0.65 | 14/24 | −0.04 dB |
| Flat graphics (24) | 100% | 0.27 → 0.63 | 0.67 → 0.56 | 12/24 | −0.94 dB |

Paintings all improve at 25%, but at 100% R overshoots greatly to 8.5× and the texture map error actually gets worse. Brush paintings have large mid-band energy, so a constant fitted to photos goes too far. Illustrations improve in 10/12 at 25%, but R barely rises, 0.07 → 0.11, so the effect is small. Flat graphics (logos · UI · line art · documents) improve in only about half. Neither is recommended.

**What the eye sees.** The numbers are closer to the ground truth at 100%, but in the comparison images (`research/results/texture_samples/`, from left: ground truth · texture off · texture on), 100% mostly looks like noise. In scenes with dense texture (a motorcycle engine, a steel tower), fine dots are visible even at 25%. Smooth surfaces such as faces and skin in portraits gain a subtle texture at 20–30%. So the guidance text recommends 20–30% for photos · paintings.

### 9.9 Display P3

Measured in real Chromium (Playwright) (`tests/p3_browser.js`). P3 primary patches are drawn on Chromium's display-p3 canvas and made into JPEG · PNG (the browser encoder attaches a P3 profile), fed into the app, upscaled 2× → saved as PNG → the saved file is read back into a P3 canvas, and the color at the center of each patch is checked.

| Patch (P3 values) | v1.0.0 saved result | v1.1.0 saved result |
|---|---|---|
| Red 255, 0, 0 | 234, 51, 35 | 255, 0, 0 (JPEG input 254, 0, 0) |
| Green 0, 255, 0 | 117, 251, 76 | 0, 255, 0 (JPEG input 0, 255, 1) |
| Gray 128, 128, 128 | 127, 127, 127 | 127, 127, 127 |

The gray 127 is the value of the test image itself. The Display P3 ICC profile generated by the app was also checked with LittleCMS (Pillow ImageCms). Converting P3 (200, 100, 50) · (60, 180, 220) to sRGB gives the same values as computing with the primaries matrix. JPEG · PNG with an sRGB profile attached and dataset files without a profile were detected as sRGB, and the P3 test images and v1.1.0 saved files as wide gamut.

### 9.10 Perceived sharpness

The 23 validation images (pixel art excluded) × 3 conditions (4× clean, 4× degraded, 8× degraded) = 70 cases are upscaled with the same settings as 「Auto」, comparing 「Accuracy」 and 「Perceptual」 (`research/pop/pop_probe.js`, `research/results/pop_val.json`). The error is $`|\ln A| + |\ln C|`$ from [4.21](#421-perceived-sharpness), which counts being blunter than the ground truth and overshooting it equally. Halo is the mean amount by which the result exceeds the ground-truth neighborhood range (radius = scale) around ground-truth edges (within 2× the scale); flat-area noise is the high-frequency RMS of the result where the ground truth is flat (bottom 30% of slope).

| Perceptual 100% | Error (Accuracy → Perceptual) | Improved | Edge slope A | Local contrast C | Halo | Flat-area noise | PSNR | SSIM |
|---|---|---|---|---|---|---|---|---|
| Photos, 9 cases | 0.303 → 0.226 | 6/9 | 0.849 → 0.949 | 0.967 → 1.088 | +0.16 levels | ×1.09 | −0.22 dB | −0.0003 |
| Paintings · illustrations · manga, 26 cases | 0.676 → 0.529 | 20/26 | 0.731 → 0.825 | 0.865 → 0.983 | +0.12 levels | ×1.16 | −0.16 dB | +0.006 |
| Line art · documents · UI · logos, 35 cases | 0.621 → 0.559 | 20/35 | 0.865 → 0.949 | 0.982 → 1.078 | +0.52 levels | ×0.95 | −0.06 dB | +0.012 |
| All 70 cases | 0.600 → 0.505 | 46/70 | 0.813 → 0.903 | 0.937 → 1.044 | +0.33 levels | ×1.05 | −0.12 dB | +0.008 |

| Strength (all 70 cases) | Error | Improved | Halo | Flat-area noise | PSNR | SSIM |
|---|---|---|---|---|---|---|
| 50% | 0.526 | 49/70 | +0.22 levels | ×1.00 | −0.06 dB | +0.006 |
| 100% | 0.505 | 46/70 | +0.33 levels | ×1.05 | −0.12 dB | +0.008 |
| 150% | 0.497 | 46/70 | +0.40 levels | ×1.09 | −0.17 dB | +0.008 |

Edge slope and local contrast rise toward the ground truth, while halo and flat-area noise barely increase. Photos and pictures improve in two thirds or more of the cases, but line art, documents, UI and logos improve in 20/35, just over half: their edges are already steep, so local contrast easily overshoots the ground truth. Comparison images are in `research/results/pop_samples/` (from left: ground truth · Accuracy · Perceptual 50% · 100% · 150%; 4× degraded, and a 500px ground truth halved and upscaled 2×).

At first, 「Perceptual」 also added fine grain (texture revival, 25% at strength 100%). The texture metric improved (high-frequency energy ratio R 0.13 → 0.30), but flat-area noise rose 4.0× and SSIM dropped by 0.044 (`pop_val_grain.json`), and dots showed on smooth surfaces. So 「Perceptual」 raises only edges and local contrast, and fine grain is chosen separately with the 「Texture revival」 slider.

---

## 10. Strengths

**Restoration honest to the information.** In the restoration stage it creates no new extremes (cell range) and fits the result so that shrinking it back gives the original (consistency). Corrections on top are added only as much as the user turns them on.

**Auto that knows the kind.** It decides by 9-class classification (92.9%), a search over 1.52 trillion settings, and measurements taken on each image. It writes on screen what it based the decision on.

**Restoration finished at the original resolution.** DCT restoration that measures and removes the noise spectrum, JPEG restoration that respects the file's quantization tables, and RL deconvolution are done before upscaling.

**Graphics with living lines.** Directional smoothing and signed-distance redrawing produce sharp lines without staircases. For pixel art, each subsample picks only colors from the original palette, and new colors appear only in the antialiasing of contours.

**Honest user settings.** Checkboxes and sliders work exactly as set (for the exception right after opening an image, see [11. Limitations](#11-limitations)). Only the 「Auto」 button automates, and the indicator merely reports measurement results.

**Not bound by size.** With the tile cache and per-tile-row PNG compression, results of hundreds of millions of pixels are saved without holding the whole uncompressed result in memory. Auto scale picks only scales that can be processed to the end within the memory budget.

**Screen = saved file.** With the same kernel, margins and global grids, what you see on screen is exactly what is saved.

**Band-free 8 bits.** Coordinate-determined dither nearly halves the banding metric while leaving exact colors, grays and tile seams as they are.

**Phone colors kept intact.** It reads the original profile, processes Display P3 photos as P3, and saves with a self-generated P3 profile and cICP attached.

**Privacy and portability.** No server and no installation, and images never leave the device. It is a single file, so it is easy to keep and move.

---

## 11. Limitations

**It does not invent.** Detail not recorded in the original (a single strand of hair, small text) cannot be brought back. It does not paint in plausible shapes the way neural upscaling does, so a very blurry original stays blurry even when enlarged.

**Texture revival is grain.** It matches the amount of texture, but the position and shape are invented noise, so raising the strength makes it look like noise. At 100% paintings overshoot greatly. For illustrations the effect is small, and flat graphics improve only about half the time at any strength. It is recommended only at low strength for photos and paintings ([9.8](#98-texture-revival)).

**「Perceptual」 adds no information.** It redraws the edges and contrast the source holds so they stand out, but creates no detail the source lacked. On images whose edges are already steep, such as line art, documents, UI and logos, only a little over half of the cases improve (20/35). A blurry result, such as a very small source upscaled a lot, stays blurry with 「Perceptual」 ([9.10](#910-perceived-sharpness)).

**Logos and comics only for JPEG input.** They did not pass the adoption rule on re-measured clean input, so large-scale restoration search values are not used for clean input such as PNG. An image that was originally JPEG and was re-saved as PNG is treated as clean input even if it has JPEG artifacts.

**Illustrations not adopted.** Results were mixed across conditions, so large-scale restoration search values are not used. There is room to look again with other degradations (strong noise, different blur, originally sharp originals) and larger images.

**Search range.** The search is based on small patches (64px input) and 3 images per kind. Sharpening amount, overshoot and the like tended to stick to the ends of their ranges, and widening the ranges gained within +0.1 dB.

**Auto scale looks only at memory.** There is no processing time limit. At 8× the result is hundreds of megapixels. The memory budget is an estimate of device memory, so 「guaranteed」(보장) is relative to this budget. This is because browsers do not report remaining memory accurately.

**Tonal range of 8-bit input.** Input is read as 8 bits through the browser canvas. Dithering and 16-bit saving only preserve the new intermediate values created by upscaling; they cannot remove bands that were already in the original (especially at 1× or on very flat areas). With dither off, 8-bit saving is simple rounding, so bands widen by the scale.

**Color management.** Wide gamut is preserved only up to Display P3. Originals with colors beyond P3, such as Adobe RGB · BT.2020 · ProPhoto, are reduced into P3, and the saved PNG gets a P3 profile. This is because canvases offer only sRGB and Display P3 as working color spaces. Still, the reduction is smaller than reading as sRGB (in CIE xy area, sRGB covers 53% of the BT.2020 triangle and P3 72%). Preserving the original profile as is would require reading without color conversion, processing in that encoding and reattaching the profile, so it is left as a separate feature. Browsers that do not support P3 canvases (e.g. current Firefox) read as sRGB as before. Pasted images are always treated as sRGB. If an AVIF keeps color information only inside the AV1 bitstream (sequence header) without a `colr` box, it is not read and the file is treated as sRGB (common encoders such as libavif write `colr`). Browsers without `DecompressionStream` (Safari 15.2–16.3) cannot decompress the compressed iCCP profile of a PNG, so PNGs that signal wide gamut only by iCCP without cICP are treated as sRGB. 「Auto」's analysis (classification · detail measurement · color auto) receives even P3 images from the browser as sRGB values when measuring, so it stays on the sRGB basis the classifier was trained on. The canvas saving path of browsers without streaming compression does not attach the P3 profile directly.

**Compression ratio assumption.** Auto scale counts the saved PNG's compression ratio as the maximum measured per scale on 47 images of the data plus a 10% margin (with 16 bits and texture on, 0.934 × 1.1, 102% of the uncompressed size). Images with much more noise than the 47 compress less than this assumption, so memory may run short during saving even at a 「guaranteed」 scale.

**Missing data.** The RGBA cache for classifier training (about 1.3 GB) and the DCCI grid experiment data (about 1.3 GB) are not in the repository because of their size. Both can be regenerated from the `data/ds` originals.

---

## 12. Changelog

### v1.2.2 — Polishing 「Auto」 and the buttons

- Pressing 「Auto」 in the scale row now moves the scale right away to the largest one guaranteed with the current settings (before, it moved only when the 「Auto」 button was pressed). The 「Auto」 button still searches again with each scale's own settings.
- In 「Goal」, the marker behind 「Accuracy」 and 「Perceptual」 now slides to the pressed option like a spring (it overshoots once and settles in about 0.5 s; with reduced motion it moves at once). Pressing another scale releases 「Auto」.
- Pressing 「Auto」 now shows its progress as a darker orange that fills the button from left to right. It reaches 100% 0.7 s after the analysis and measurements finish, and **Run upscale** stays disabled until then (so it is not pressed the instant they end). Choosing a preset yourself after 「Auto」 now says that 「Auto」 was released, and any measurement still running stops so it cannot overwrite that choice.

### v1.2.1 — Name and version display

- The app title is now **Upscaly: 무손실 지향 확대기** (Upscaly: lossless-oriented upscaler), and the version (v1.2.1) is shown next to the name in the top bar. On narrow screens (720px or less) only "Upscaly" and the version are shown.
- Computation and saved results are the same as in v1.2.0.

### v1.2.0 — Two goals: accuracy and perception

- **「Goal: Accuracy · Perceptual」.** The default 「Accuracy」 upscales as close to the original as before (the output is byte-identical to v1.1.1). 「Perceptual」 raises edge slope and local contrast to the ground-truth level so the result looks crisper. It never exceeds the surrounding brightness range, so no white fringes appear. On 70 validation cases the error against the ground truth fell from 0.600 to 0.505, with flat-area noise ×1.05 and PSNR −0.12 dB ([2.9](#29-accuracy-and-perceived-sharpness-are-different-goals), [9.10](#910-perceived-sharpness)).
- Perceptual strength 25–150%; presets and 「Auto」 do not change this choice. Auto scale counts the margin, memory and saved-PNG compression ratio of 「Perceptual」.

### v1.1.1 — Fixes for issues found in code review

- **Transparent images with staged upscaling.** In the intermediate stages, alpha also goes through the same direction weights, cell range and inter-stage sharpening as color. At DCCI 4×, semi-transparent edge color error dropped from at most 19–38 levels to 1–4 levels, and the tile seams of the logo preset at 4× (at most 206 levels) disappeared.
- **Initial value of 「Upscale in stages」.** If you change the checkbox or press 「Auto」 before the measurement right after opening an image finishes, the late-finishing measurement does not overwrite your choice.
- **Blank canvas check.** If the source canvas is entirely transparent, the original is read directly in pieces to see whether any pixel is visible, and canvas saving finds one visible pixel in the result and checks that spot. The canvas limit is detected even in images whose bottom-right is transparent to begin with or where only one dot is visible.
- **Bright colors in texture revival.** The brightening ratio is limited to the headroom of the largest channel, so the hue of bright, highly saturated colors does not change. The constant was refitted accordingly (5.915 → 6.256), and the measurements were corrected to the same conditions as 「Auto」 and redone. Texture overshooting 1.8× in gamma mode, with 「Compute in linear light」 off, was also fixed.
- **Gamut detection.** Even if more than 2 MB of metadata or boxes (JPEG extended XMP, PNG text chunks, AVIF mdat before meta) come before the color information, it follows the headers to find it. For AVIF it looks at the color information (colr) associated with the primary image rather than a thumbnail or auxiliary image, and for PNG it looks at cICP first regardless of chunk order, but if the primaries are unspecified (2), it decides by the iCCP profile. Wide-gamut files are read via a Data URL `<img>` even when Blob URLs are blocked, preserving their colors.
- **Budget after 「Auto」.** 「Auto」 applies each scale's 「Auto」 settings starting from the largest scale and picks the first guaranteed scale. Choosing 8× for 1200×900 with nearest · texture on and then running short of source preparation memory no longer happens. Staying at a smaller scale when settings split into cheaper and more expensive ones depending on scale also no longer happens (UI image 4000×3000, current scale 1.25×, three-stage staged upscaling: 「No guarantee」 → 2×).
- **Auto scale memory calculation.** It counts the compressed PNG that piles up while saving (maximum compression ratio per scale measured by enlarging 47 images of the data at 1.1–4×, plus a 10% margin) and the other prepared originals left in the cache. So the chosen scale may be lower for large photos (with a 600 MB budget, 4000×3000 gets 2× rather than 4×). Changing the save bit depth or texture also recomputes the notice.

### v1.1.0 — Band-free 8 bits, phone colors, texture revival, logo revalidation

- **8-bit dithering (on by default).** When converting to 8 bits, fine coordinate-determined dither is mixed in, reducing bands in enlarged gradients (banding metric −44%). It can be turned off with the 「8-bit dithering」(8비트 디더링) checkbox. 16-bit output is byte-for-byte identical to v1.0.0.
- **Display P3 preservation.** If the color profile of the original file is wider than sRGB, it is read into a P3 canvas, the screen is also drawn in P3, and a Display P3 profile (cICP · iCCP) is attached to the saved PNG. P3 reds and greens are no longer squeezed as before.
- **Texture revival (off by default).** A new slider adds missing fine texture matched to the texture strength of the original. 20–30% is recommended for photos and paintings.
- **Logo · comic revalidation.** The clean-input validation error was fixed and they were re-measured. Logos now use large-scale restoration search values for JPEG input, and comics use them only for JPEG input.
- The README was rewritten in paper style.

### v1.0.0 — First version moved to Claude Code

- The project, which had been developed locally with Claude, was moved into the repository. The app (`upscaler.html`), baseline, tests, research scripts and results, and data (567 originals, search patches) were uploaded as they were.
- Conversion results are byte-for-byte identical to the baseline across 134 combinations.

---

## 13. References

1. D. Zhou, X. Shen, W. Dong. "Image zooming using directional cubic convolution interpolation." *IET Image Processing* 6(6), 2012.
2. R. Keys. "Cubic convolution interpolation for digital image processing." *IEEE Trans. ASSP* 29(6), 1981.
3. D. P. Mitchell, A. N. Netravali. "Reconstruction filters in computer graphics." *SIGGRAPH* 1988.
4. C. E. Duchon. "Lanczos filtering in one and two dimensions." *J. Applied Meteorology* 18(8), 1979.
5. W. H. Richardson. "Bayesian-based iterative method of image restoration." *JOSA* 62(1), 1972.
6. L. B. Lucy. "An iterative technique for the rectification of observed distributions." *Astronomical Journal* 79, 1974.
7. C. Tomasi, R. Manduchi. "Bilateral filtering for gray and color images." *ICCV* 1998.
8. K. Dabov, A. Foi, V. Katkovnik, K. Egiazarian. "Image denoising by sparse 3-D transform-domain collaborative filtering." *IEEE TIP* 16(8), 2007.
9. A. Zakhor. "Iterative procedures for reduction of blocking effects in transform image coding." *IEEE TCSVT* 2(1), 1992.
10. Y. Yang, N. P. Galatsanos, A. K. Katsaggelos. "Regularized reconstruction to reduce blocking artifacts of block discrete cosine transform compressed images." *IEEE TCSVT* 3(6), 1993.
11. K. He, J. Sun, X. Tang. "Guided image filtering." *IEEE TPAMI* 35(6), 2013.
12. J. Immerkær. "Fast noise variance estimation." *CVIU* 64(2), 1996.
13. V. A. Marchenko, L. A. Pastur. "Distribution of eigenvalues for some sets of random matrices." *Mat. Sbornik* 72(4), 1967.
14. J. Bigün, G. H. Granlund. "Optimal orientation detection of linear symmetry." *ICCV* 1987.
15. R. Achanta, S. Hemami, F. Estrada, S. Süsstrunk. "Frequency-tuned salient region detection." *CVPR* 2009.
16. Z. Wang, A. C. Bovik, H. R. Sheikh, E. P. Simoncelli. "Image quality assessment: from error visibility to structural similarity." *IEEE TIP* 13(4), 2004.
17. G. D. Finlayson, E. Trezzi. "Shades of gray and colour constancy." *Color and Imaging Conference* 2004.
18. J. van de Weijer, T. Gevers, A. Gijsenij. "Edge-based color constancy." *IEEE TIP* 16(9), 2007.
19. H. H. Bauschke, J. M. Borwein. "On projection algorithms for solving convex feasibility problems." *SIAM Review* 38(3), 1996.
20. L. H. Thomas. "Elliptic problems in linear difference equations over a network." Watson Sci. Comput. Lab. Report, Columbia University, 1949.
21. I. Kutskir. "Fastest Gaussian blur (in linear time)." 2013. Blog post.
22. IEC 61966-2-1:1999. Multimedia systems and equipment — Colour measurement and management — Part 2-1: Default RGB colour space — sRGB.
23. J. Jimenez. "Next generation post processing in Call of Duty: Advanced Warfare." SIGGRAPH 2014 Advances in Real-Time Rendering talk. (interleaved gradient noise)
24. D. J. Field. "Relations between the statistics of natural images and the response properties of cortical cells." *JOSA A* 4(12), 1987.
25. A. van der Schaaf, J. H. van Hateren. "Modelling the power spectra of natural images: statistics and information." *Vision Research* 36(17), 1996.
26. International Color Consortium. "ICC.1:2022 Image technology colour management — Architecture, profile format, and data structure." 2022.
27. W3C. "Portable Network Graphics (PNG) Specification (Third Edition)." 2025. (cICP chunk)
28. SMPTE EG 432-1:2010. Digital Source Processing — Color Processing for D-Cinema. (P3 primaries)
29. R. A. Ulichney. *Digital Halftoning*. MIT Press, 1987.
