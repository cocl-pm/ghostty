# Devanagari Rendering Limitations

## Current State
The `cell_width` field has been plumbed through to CellText structs and vertex shaders
(Metal + OpenGL) to enable future glyph boundary clamping. The upstream Terminal.zig
widening logic is unchanged — non-zero-width-in-grapheme codepoints widen grapheme
clusters to 2 cells via `desired_wide = .wide`.

Requires `grapheme-width-method = unicode` in ghostty config.

## Known Issues

### Standalone vowels (आ, औ, etc.) get overlapped by adjacent glyphs
- **Cause:** GPU renders text glyphs as instanced draw calls in cell order (left-to-right).
  With premultiplied alpha blending, the next cell's glyph overwrites the previous cell's
  overflow. Standalone vowels like आ (aa) and औ (au) have glyphs wider than 1 cell, but
  their rightward overflow gets painted over by the next glyph.
- **Impact:** Minor visual cramping on standalone wide vowels.
- **Possible fixes:**
  - Two-pass rendering (separate pass for overflow regions)
  - Rendering order changes (right-to-left would fix right-overflow but break left-overflow like ि)
  - Fragment shader logic to avoid overwriting occupied adjacent pixels
- **Status:** Accepted limitation for now.

### Extra spacing with Mc matras
- **Cause:** Spacing marks (Mc) like ा, ी, े, ो trigger `desired_wide = .wide`, expanding
  clusters to 2 cells. Some clusters only need ~1.5 cells, so 2 cells creates visible
  extra space.
- **Impact:** Words like "हालाँकि", "बारे", "में", "प्रचलित" show slightly extra spacing.
- **Possible fix:** Distinguish Mc marks that truly need 2 cells (right-extending like ा, ी)
  from those that don't (above-only like े, ै). This is complex and font-dependent.
- **Status:** Accepted.
