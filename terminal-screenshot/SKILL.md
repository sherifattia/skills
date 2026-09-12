---
name: terminal-screenshot
description: Capture polished, reproducible terminal screenshots and readable GIFs from real CLI runs. Use for README heroes, PR evidence, documentation, or any visual proof of command-line behavior; never fabricate output.
compatibility: Requires VHS and FFmpeg. Gifsicle is recommended for GIF optimization.
---

# Terminal Screenshots and GIFs

Show the product working in a terminal that looks native, clear, and trustworthy. A reader should understand the command and result in one viewing.

## Non-negotiable

- Run the real executable and let the real renderer produce the visible output. A deterministic harness may control external dependencies, but never recreate product output with `echo`, fixtures, or hand-authored terminal text.
- Show the exact command a user should enter, such as `npx @scope/package ...`. Hide capture scripts, scenario paths, environment setup, and other recording machinery before clearing the terminal.
- Keep the output quiet and relevant. Remove unrelated warnings and prompts, but never hide behavior the user needs to judge.
- Capture VHS sessions sequentially. Concurrent sessions can corrupt glyphs or retain stale cells.

## Visual default

- Present every CLI capture inside a macOS-style terminal: dark blue-gray surface (`#1e2430`), high-contrast text (`#f2f2f2`), Menlo or another native monospace font, traffic-light controls, generous padding, and rounded corners.
- Keep the canvas outside the terminal transparent so the asset works on GitHub light and dark themes.
- Use a subtle near-black edge (`#0f141d`), never a white or light border. The terminal surface must remain visibly distinct from a dark page.
- Prefer roughly 1460×900 for a README hero, 20 px type, 32 px padding, 36 px transparent margin, and a 48 px title bar. Adjust height to the real output; increase width before accepting awkward wrapping or clipping.
- Keep all content inside the readable text column. Structural prefixes such as `│ ` stay fixed; wrapped continuation lines align after the prefix instead of entering the border.

For reliable transparent corners, record a square terminal against a chroma-key margin, then apply a rounded alpha mask in post-processing. Do not rely on antialiased VHS corners against the key color: they can leave a colored fringe. Use about a 16 px radius with a 1 px dark edge.

## Human pacing

- Pace an animated demo for one comfortable viewing, usually 15–25 seconds.
- Leave 0.5–1 second after the command, enough time to read each meaningful phase, and 4–5 seconds on the settled result.
- Favor comprehension over speed. If a reviewer must replay the GIF to parse the command, progress, or result, slow it down.
- Optimize the finished GIF with `gifsicle -O3` or an equivalent lossless optimizer without removing readable holds.

## Capture and verify

Use VHS for the real session. When a PNG is needed, extract it from the settled GIF with FFmpeg; VHS's direct `Screenshot` command can omit unchanged cells.

```sh
ffmpeg -loglevel error -y -sseof -0.1 \
  -i /private/tmp/terminal-demo.gif \
  -frames:v 1 path/to/output.png
```

Before committing:

1. Watch the GIF once at its intended README or PR size.
2. Inspect the opening, a representative middle frame, and the final frame at original resolution.
3. Composite a frame over white and GitHub dark (`#0d1117`) backgrounds. Confirm transparent corners, clear window separation, and no halo.
4. Confirm the visible command is the real user command, the output comes from a real run, important text is not clipped or wrapped into chrome, and no secret, cursor artifact, setup command, or warning remains.
5. Commit the tape and capture script with the optimized media so the evidence is reproducible.
