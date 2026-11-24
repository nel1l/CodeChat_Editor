# YAML metadata block support
============================

This specification defines how a leading YAML-style metadata block moves
between Markdown, HTML, and the CodeChat Editor. It does not parse, validate, or
reformat YAML syntax. Everything not described here is unchanged, including
ordinary Markdown, HTML, and non-YAML script handling.

The conversion paths are `pulldown-cmark` in
[processing.rs](../server/src/processing.rs), `htmd` in the same file, and
TinyMCE configured in [tinymce-config.mts](../client/src/tinymce-config.mts).
Server translation and sanitization in
[translation.rs](../server/src/translation.rs) and
[processing.rs](../server/src/processing.rs) must preserve this data block
without making executable scripts permissible.

## 1. Markdown-to-HTML (`pulldown-cmark`)

### Recognition

1. Enable `Options::ENABLE_YAML_STYLE_METADATA_BLOCKS` for Markdown-to-HTML
   conversion.
2. Recognize metadata only when the first line of the Markdown document is
   exactly `---` and a later line consists exactly of `---` or `...`.
   Delimiters are standalone lines; indentation or trailing characters make a
   line ineligible as a delimiter.
3. The first eligible closing delimiter ends the metadata block. Text between
   the delimiters is the payload. An empty payload is valid.
4. A delimiter-looking line anywhere other than the document's first line
   does not begin a metadata block. It retains its existing CommonMark meaning,
   including a thematic break or a Setext heading underline.
5. If a leading `---` has no eligible closing delimiter, it is not a metadata
   block; the input falls back to existing Markdown parsing.

### Output

1. Render a recognized metadata block as one
   `<script type="application/yaml">` element before the HTML for the remaining
   Markdown.
2. Place the payload directly between the opening and closing tags. Add no
   newline immediately after the opening tag or immediately before the closing
   tag. Do not HTML-entity-escape payload `&`, `<`, or `>` characters, except
   for the closing-sequence protection specified in
   [Section 3](#3-escaping-delimiters-and-edge-cases).
3. Do not include either Markdown delimiter in the script payload. Preserve
   payload characters and line breaks; the closing delimiter (`---` or `...`)
   is not part of the payload.

## 2. HTML-to-Markdown (`htmd`)

### Recognition

1. Treat a `<script>` as a YAML metadata candidate only when it has a `type`
   attribute whose value, after trimming ASCII whitespace, equals
   `application/yaml` using ASCII-case-insensitive comparison.
2. Ignore other attributes when deciding whether the element is a candidate.
   They do not become Markdown metadata.
3. Only a candidate that is the first non-whitespace node in the document
   fragment is metadata. Whitespace-only text nodes before it do not count as
   content. At most one leading candidate is metadata.
4. A non-leading YAML-typed script is not metadata and is omitted like other
   script elements. Untyped scripts and scripts with any other type, including
   JavaScript types, are omitted as they are today; their contents are not
   converted to Markdown.
5. Decode the escaped payload representation described in
   [Section 3](#3-escaping-delimiters-and-edge-cases) before emitting Markdown.

### Output

1. Emit a recognized leading candidate as a YAML-style metadata block. Use
   `---` for both delimiters, including when the Markdown-to-HTML input used
   `...`.
2. Emit the opening delimiter, one line break, the exact decoded payload, one
   line break, and the closing delimiter. The line breaks framing the payload
   are not part of it.
3. If Markdown content follows the closing delimiter, separate it from that
   delimiter with one blank line. Do not copy the candidate script's attributes
   into Markdown.
4. An empty script produces an empty metadata block: the two delimiters occupy
   consecutive lines, with no payload line between them.

## 3. Escaping, Delimiters, and Edge Cases

1. Delimiter framing is separate from payload. The Markdown-to-HTML opening
   tag is immediately followed by the payload, and the payload is immediately
   followed by the closing tag. HTML-to-Markdown adds exactly the line breaks
   needed around the payload and delimiter lines; it must not trim payload
   whitespace.
2. A payload's leading newline, trailing newline, or trailing blank lines are
   payload characters and must survive a round trip. For example, a payload
   ending in two line feeds has two line feeds before the closing `---` in
   Markdown, in addition to the one line feed that frames the closing
   delimiter.
3. `---\n---` and `<script type="application/yaml"></script>` both represent
   an empty payload. They do not represent a payload containing a newline. Line endings within the payload normalize to LF (`\n`) during HTML parsing; CRLF (`\r\n`) and lone CR (`\r`) sequences are not preserved across a round trip.
4. HTML script contents are parsed as raw text, where a case-insensitive
   `</script` sequence would terminate the element. Use this reversible
   encoding for every YAML payload, in both directions:
   * Before writing the payload into a script, double each literal backslash
     (`\` becomes `\\`), then replace the slash in each case-insensitive
     `</script` sequence with `\/`. Preserve the original letter casing.
   * When reading a candidate script, scan left to right. Convert an encoded
     `\/` in a `</script` sequence back to `/`, and convert each encoded `\\`
     back to `\`. Do not change any other backslashes or payload characters.
   * This ordering makes an original literal backslash adjacent to
     `</script`, or an original `\/script` sequence, distinguishable from the
     inserted escape. The transformation is not YAML parsing or YAML
     reformatting.
5. Apart from the reversible closing-sequence encoding above, do not escape
   `&`, `<`, or `>` in the payload. In particular, character references are
   not a substitute for the closing-sequence encoding: script raw text does
   not decode them.

## 4. Editor Pipeline Integration and Examples

### Pipeline requirements

1. The Markdown renderer in
   [processing.rs](../server/src/processing.rs) must enable the YAML metadata
   option and preserve the rendered data block through the normal server
   processing path.
2. Server HTML cleaning and transformations in
   [processing.rs](../server/src/processing.rs) and
   [translation.rs](../server/src/translation.rs) must retain a
   `type="application/yaml"` data script and its text without changing the
   payload, while stripping all other attributes (such as `src` or event
   handlers). They must not allow untyped or executable scripts as a side
   effect.
3. The `htmd` conversion path must recognize and emit only a leading YAML
   candidate as defined above. Script elements of other types remain ignored.
4. TinyMCE configuration in
   [tinymce-config.mts](../client/src/tinymce-config.mts) must preserve only the
   YAML data script, its `type` attribute, and its text payload through editor
   load and serialization, stripping any other attributes. This permission must
   be limited to YAML data scripts; it must not make JavaScript or arbitrary
   scripts executable or preservable.
5. An unchanged document must survive Markdown → HTML → editor → HTML →
   Markdown with the metadata payload and its position preserved, subject only to delimiter normalization, line-ending normalization to LF, and the reversible closing-sequence encoding.

### Examples

The outputs below show the conversion result for the indicated fragment.
`<script>` payload escapes shown here are literal characters in the HTML text.

#### Markdown to HTML

**A leading metadata block with the `---` closer**

Input:

```markdown
---
title: Example
tags:
  - one
---

# Heading
```

Output:

```html
<script type="application/yaml">title: Example
tags:
  - one</script>
<h1>Heading</h1>
```

The block is at the start of the document and closes on its own line. The
script's opening tag and payload, and its payload and closing tag, have no
framing newline between them.

**The `...` closer and an empty block**

Input:

```markdown
---
title: Example
...
```

Output:

```html
<script type="application/yaml">title: Example</script>
```

The closer is normalized to an HTML closing tag; it is not included in the
payload.

The empty block has no payload line:

Input:

```markdown
---
---
```

Output:

```html
<script type="application/yaml"></script>
```

The empty payload produces an empty script element, with no newline between
its tags.

**Special characters and an embedded closing sequence**

Input:

```markdown
---
symbols: "& < >"
literal: </SCRIPT>
---
```

Output:

```html
<script type="application/yaml">symbols: "& < >"
literal: <\/SCRIPT></script>
```

The ordinary `&`, `<`, and `>` remain unescaped. The case-insensitive
`</SCRIPT` is protected so it cannot end the HTML script element early.

**Trailing blank lines in the payload**

Input:

```markdown
---
value: x

---
```

Output:

```html
<script type="application/yaml">value: x

</script>
```

The blank line belongs to the payload and remains immediately before the
closing tag; it is not framing whitespace to be trimmed.

**A non-leading delimiter and an unclosed leading delimiter**

Input:

```markdown
Before

---
title: not metadata
---

After
```

Output:

```html
<p>Before</p>
<hr />
<h2>title: not metadata</h2>
<p>After</p>
```

The delimiters are not at the start of the document, so they retain their
ordinary Markdown meaning. A leading `---` without a later eligible closer
also falls back to ordinary Markdown parsing rather than producing a metadata
script.

An unclosed leading delimiter has this result:

Input:

```markdown
---
title: unfinished
```

Output:

```html
<hr />
<p>title: unfinished</p>
```

Without a closing delimiter, the opening line is parsed as a thematic break and
the following text remains ordinary Markdown.

#### HTML to Markdown

**A leading YAML script**

Input:

```html
<script type="application/yaml">title: Example</script><p>Hello</p>
```

Output:

```markdown
---
title: Example
---

Hello
```

The leading typed script becomes metadata, and the following Markdown content
is separated from the closing delimiter by one blank line.

**An empty script**

Input:

```html
<script type="application/yaml"></script>
```

Output:

```markdown
---
---
```

No payload line is inserted between the delimiters.

**The normalized closer and special characters**

Input:

```html
<script type=" APPLICATION/YAML ">symbols: "& < >"</script>
```

Output:

```markdown
---
symbols: "& < >"
---
```

ASCII whitespace around the type value is trimmed and the comparison ignores
ASCII letter case. Payload characters are not entity-decoded or reformatted.

**An escaped embedded closing sequence**

Input:

```html
<script type="application/yaml">literal: <\/SCRIPT></script>
```

Output:

```markdown
---
literal: </SCRIPT>
---
```

The encoded slash prevents the HTML parser from closing the script early; the
HTML-to-Markdown path removes only the escape inserted by this specification.

**A non-leading YAML script and non-YAML scripts**

Input:

```html
<p>Before</p><script type="application/yaml">hidden: true</script><p>After</p>
<script>run()</script><script type="text/javascript">run()</script>
```

Output:

```markdown
Before

After
```

Only a leading YAML-typed script is metadata. A non-leading YAML script,
untyped script, and JavaScript script are omitted, including their contents.