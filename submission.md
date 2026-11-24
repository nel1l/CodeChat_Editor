Project 2 submission
====================

Team name: yaml support

The specification consists of the file [yaml-support-spec.md](yaml-support-spec.md).

Final submission
----------------

### Assem Temirbay

GitHub userid: 122988929

#### Contributions

Authored the initial YAML metadata block specification (`yaml-support-spec.md`), refined edge cases and attribute stripping rules across three LLM review rounds, and initialized the team's `submission.md` tracking document.

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | 2 | Fixed CommonMark example where `---` directly following text formed a Setext `<h2>` instead of `<hr />`, and clarified line-ending normalization to LF. |
| 2 | 1 | Clarified that only the `type="application/yaml"` attribute and text payload survive sanitization and serialization while other attributes are stripped. |
| 3 | 5 | No changes made; findings prescribed internal implementation details (overspecification) or trivial HTML parser edge cases. |

#### Last specification review

Prompting Copilot Fusion (`mai-code-2-flash`) with `review the spec in yaml-support-spec.md` produced the following output (but before I used gpt-luna-6 with high effort); strikethrough indicates LLM output deemed overspecified, irrelevant to the specification layer, or trivial:

> ## Findings
>
> ~~1. **High — The HTML-to-Markdown path will reformat payload text**~~
>   ~~- `yaml-support-spec.md:70` requires exact payload preservation, including whitespace and blank lines.~~
>   ~~- `processing.rs:678` converts HTML through htmd and then always `formats and word-wraps` the result.~~
>   ~~- The implementation needs a protected metadata region or must bypass `format_text` for the payload; otherwise payload lines can change despite the spec.~~
>
> ~~2. **High — Existing server sanitization explicitly removes scripts**~~
> ~~- `AMMONIA_OPTIONS` does not add `<script>`, and `processing.rs:1287` currently documents scripts as removed content.~~
>   ~~- `minify` will therefore delete the metadata block unless ammonia is updated with narrowly scoped script/type handling.~~
>
> ~~3. **High — TinyMCE configuration does not permit a YAML script**~~
> ~~- `tinymce-config.mts:149` only extends custom elements for graph/CodeChat elements.~~
>   ~~- The spec requires preservation of only leading YAML scripts, but no matching `script[type=application/yaml]` configuration or serialization hook exists.~~
>
> ~~4. **Medium — Leading-node recognition is ambiguous for comments and non-content nodes**~~
> ~~- `yaml-support-spec.md:62` says the candidate must be the first non-whitespace node, but does not specify whether HTML comments, doctypes, or TinyMCE temporary nodes count.~~
>   ~~- This needs an explicit rule to avoid differing behavior between html5ever, htmd, and TinyMCE.~~
>
> ~~5. **Medium — “ASCII whitespace” needs an exact character definition**~~
> ~~- `yaml-support-spec.md:56` uses trimming but does not enumerate the ASCII whitespace characters.~~
>   ~~- The safest implementation-compatible definition is `HTAB`, `LF`, `FF`, `CR`, and `SPACE`; this should be stated explicitly to prevent locale or Unicode trimming.~~

#### Implementation review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | (TBD) | (TBD) |

#### Last implementation review

(TBD — add LLM, prompt, and final implementation review output after implementing the feature)

### [Team member 2 name]

GitHub userid: [team-member-2-github]

#### Contributions

(One sentence summarizing Team member 2's contributions)

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last specification review

(LLM and prompt used)

(LLM output, possibly with strikethrough)

#### Implementation review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last implementation review

(LLM and prompt used)

(LLM output, possibly with strikethrough)

### [Team member 3 name]

GitHub userid: [team-member-3-github]

#### Contributions

(One sentence summarizing Team member 3's contributions)

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last specification review

(LLM and prompt used)

(LLM output, possibly with strikethrough)

#### Implementation review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last implementation review

(LLM and prompt used)

(LLM output, possibly with strikethrough)

### [Team member 4 name]

GitHub userid: [team-member-4-github]

#### Contributions

(One sentence summarizing Team member 4's contributions)

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last specification review

(LLM and prompt used)

(LLM output, possibly with strikethrough)

#### Implementation review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | | |

#### Last implementation review

(LLM and prompt used)

(LLM output, possibly with strikethrough)