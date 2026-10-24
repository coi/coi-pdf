# PDF for Coi

Write PDF files from drawing commands, in [Coi](https://github.com/coi-lang/coi): pages of paths,
pictures and text, with TrueType fonts embedded so the text looks right and can be copied.

```bash
coi add @coi/pdf
```

```tsx
import "@coi/pdf";

component Export {
    mut Pdf::Writer pdf = Pdf::Writer(&written);

    def save() : void {
        pdf.begin();
        pdf.addFont("body", "/fonts/inter-latin-400.ttf");
        pdf.addFont("body", "/fonts/inter-latin-ext-400.ttf");   // the same name: a fallback for characters the first lacks
        pdf.addPage(595.0, 842.0);                               // A4 in points; origin top left, y down
        pdf.setFillColor(31, 41, 51, 1.0);
        pdf.setFont("body");
        pdf.drawText("Hello, Łódź", 72.0, 100.0, 12.0);         // y is the baseline
        pdf.finish();
    }
    def written(uint8[] data, string error) : void {
        if (error.isEmpty()) Files.save("hello.pdf", "application/pdf", data);
    }
}
```

## What it does

- **Paths:** `moveTo`, `lineTo`, `quadTo`, `curveTo`, `rect`, `closePath`, `stroke`, `fill`,
  `setStrokeColor`/`setFillColor` (r, g, b, alpha: transparency works), `setLineWidth`,
  `setLineCap`, `setLineJoin`, `save`/`restore`.
- **Pictures:** `drawImage` takes an `<img>` or a `<canvas>` and copies its pixels right away. Opaque
  ones go in as JPEG, the rest keep their transparency. The same `<img>` drawn again is stored once.
- **Text:** `drawText` in the font `setFont` picked. `addFont` names a TrueType file (`.ttf`,
  glyph outlines in `glyf`; not OpenType/CFF, not woff/woff2) by URL; several files under one
  name make a chain, each character drawn from the first file that has it. The files are fetched
  when the document is finished and embedded with only the glyphs used, plus a map back to the
  text, so the PDF's text can be searched and copied. Without a font it's Helvetica (in every
  reader, Latin-1 only); a file that can't be fetched or read falls back to it too.
- **Finishing:** `finish` fetches the fonts, deflates every stream (the browser's
  `CompressionStream`) and hands the file to the callback. `begin` starts the next document.

No kerning or shaping: scripts that need ligatures to read (Arabic, Indic scripts) won't come out
right. A variable font is used at its default instance: give each weight its own static file.

## License

MIT
