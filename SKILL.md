---
name: xiaohongshu-cover-maker
description: "Use when the user wants to generate a Xiaohongshu-style cover from an uploaded reference cover and new Chinese copy. Also trigger for Chinese requests such as 小红书封面创作, 小红书封面, 封面改字, or 按参考图做封面. Supports two simple modes: non-face IP covers that only need a reference cover and replacement text, and face IP covers that also need the user's portrait photo."
---

# 小红书封面创作

## Overview

Create a new Xiaohongshu-style cover by following an uploaded benchmark cover's composition, mood, and text hierarchy while replacing the copy with the user's requested text.

Important workflow principle: for Chinese cover text, prefer a two-step process. Use image generation for the visual base, icon, lighting, composition, and overall mood, then add the exact Chinese copy locally with reliable fonts and layout. Do not rely on the image model to render final Chinese text when the user needs exact wording.

If the user asks how to use this skill, read [references/simple-usage.md](references/simple-usage.md) and answer briefly in Chinese.

Keep the workflow simple. The user should only need to provide:
- A benchmark cover image.
- New title/copy.
- A portrait photo only when the cover is a face IP cover.

## Mode Decision

Choose one of two modes:

- `non_face_ip`: the benchmark cover does not use a recognizable personal face as the main IP. Examples: landscape, object, tool interface, product, abstract concept, astronaut, illustration, text-heavy poster.
- `face_ip`: the benchmark cover uses a person or face as the main visual IP. The user should provide their own portrait photo if they want the new cover to feature them.

If the mode is obvious from the reference cover, proceed. Ask one concise question only when the user has not provided a required portrait photo for `face_ip`.

## Inputs

Expected input fields:

```json
{
  "mode": "non_face_ip | face_ip",
  "reference_cover": "uploaded image",
  "portrait_photo": "uploaded image, required only for face_ip",
  "main_title": "required",
  "subtitle": "optional",
  "top_label": "optional",
  "accent_text": "optional"
}
```

Treat free-form user copy as the source of truth. If the user sends only one line of text, use it as `main_title`. If they send multiple lines, map them in order to `top_label`, `main_title`, `subtitle`, and `accent_text` when that fits the benchmark cover.

## Generation Rules

Use the `imagegen` skill / built-in image generation flow when producing the cover.

General rules:
- Preserve the benchmark cover's visual category, layout rhythm, contrast, and text hierarchy.
- Replace the benchmark text with the user's new copy, but keep exact final text rendering under local control whenever possible.
- Do not copy the benchmark image exactly; create a new cover with the same kind of impact.
- Always make the output a 3:4 vertical Xiaohongshu cover. Default to a 1080 x 1440 style composition, and explicitly say `3:4 vertical cover` in generation prompts. Do not use 9:16 unless the user explicitly asks for it.
- Prioritize readable Chinese text. Default to generating a clean visual base with blank or placeholder text areas, then locally overlay the user's exact text.
- If the cover uses very short English text or non-critical decorative text, model-rendered text is acceptable. For Chinese titles, subtitles, labels, price text, dates, names, or any copy the user explicitly provides, use local text overlay.
- When local overlay is used, match the reference cover's text hierarchy: position, scale, contrast, shadow/stroke, outline, glow, label containers, and emphasis. If a phrase is too long for one line, split it into a natural two-line title rather than shrinking it until it becomes weak.
- Do not add platform logos, watermarks, or third-party brand marks unless the user explicitly requests it and has rights to use them.

## Production Workflow

Use this workflow by default:

1. Analyze the benchmark cover:
   - mode: `non_face_ip` or `face_ip`
   - major visual blocks: top label, main title, subtitle, icon/object/person, badge, bottom label
   - background, color, contrast, depth, and text hierarchy
2. Generate or build the visual base:
   - keep the same composition rhythm and mood
   - create fresh artwork, icons, objects, or scene elements
   - leave clear space for the final text, or use neutral placeholder blocks instead of final Chinese copy
3. Add exact text locally:
   - use the user's copy verbatim
   - choose local fonts that support Chinese
   - apply bold size, stroke, shadow, outline, glow, or label containers to match the reference
   - verify no missing characters, wrong words, cropped text, or overlap
4. Save the finished cover as a local image file when possible, then visually inspect it before reporting completion.

For `non_face_ip`:
- Follow the benchmark cover's world, scene, or symbolic subject.
- Preserve major layout blocks such as top label, huge central title, accent handwriting, foreground object, or background environment.
- Generate a fresh scene that matches the theme implied by the user's new title.
- If there is a software/product/app icon in the reference, replace it with a fresh icon or object matching the requested subject. Avoid third-party logos unless the user explicitly asks and has rights.
- No user portrait is required.

For `face_ip`:
- Use the benchmark cover for composition, lighting, pose energy, text placement, and overall drama.
- Use the user's portrait as the identity reference.
- Preserve the user's face identity as much as possible while adapting outfit, lighting, background, and pose to the benchmark style.
- Avoid placing large text over the eyes, nose, or mouth unless the benchmark intentionally does that.

## Prompt Pattern

Convert the user's request into a compact production prompt.

For `non_face_ip`:

```text
Create a 3:4 vertical Xiaohongshu cover visual base inspired by the uploaded reference cover.
Mode: non-face IP.
Use the reference for layout, contrast, typography hierarchy, composition rhythm, and overall mood.
Create a fresh visual scene matching the new copy.
Leave clear blank/placeholder areas for the final local text overlay:
Top label area: <top_label or none>
Main title area: <main_title>
Subtitle area: <subtitle or none>
Accent text area: <accent_text or none>
Do not render final Chinese text inside the generated image unless the text is only decorative.
Do not copy the original artwork exactly. No watermark or platform logo.
```

For `face_ip`:

```text
Create a 3:4 vertical Xiaohongshu cover visual base inspired by the uploaded reference cover.
Mode: face IP.
Use the reference cover for composition, lighting, energy, text placement, and visual impact.
Use the uploaded portrait photo as the identity reference for the main person.
Leave clear blank/placeholder areas for the final local text overlay:
Top label area: <top_label or none>
Main title area: <main_title>
Subtitle area: <subtitle or none>
Accent text area: <accent_text or none>
Keep the face recognizable, cinematic, sharp, and expressive.
Do not render final Chinese text over the face. Final Chinese copy will be added locally.
Do not copy the original artwork exactly. No watermark or platform logo.
```

## Output

Default output:
- Generate 1 finished cover.
- If the user asks for options, generate 3 variants: `closer_to_reference`, `more_viral`, and `cleaner_personal_brand`.

After generation, briefly report:
- Which mode was used.
- Which text fields were applied.
- Whether exact text was overlaid locally.
- Where the generated cover was saved if a file path is available.
