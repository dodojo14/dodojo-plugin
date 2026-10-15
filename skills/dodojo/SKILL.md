---
name: dodojo
description: Capture, retrieve, update, organize, and revisit the user's private memories in dodojo (also written DoDoJo; 渡渡鸟 in Chinese) through the dodojo OAuth MCP server, using Markdown by default and special templates only when explicitly requested. Use only when the user names dodojo as the place to act - for example "save this to dodojo", "search my dodojo memories", reading a dodojo share link, filing into a dodojo collection, updating or trashing a memory an agent created in dodojo, setting a dodojo reminder, or sending a completion notification through dodojo. Do not use for a generic request to remember something, for the assistant's built-in memory, or for questions about the dodo bird.
---

# dodojo

Use dodojo as the user-owned memory system of record. Access it only through the
configured OAuth MCP server; never request database credentials, API keys, or a
direct database connection.

## Connect safely

- Require the `dodojo` MCP dependency. If it is unavailable, ask the user to
  connect or reauthorize dodojo, then resume the original task.
- Treat OAuth as the authorization boundary. Never bypass per-memory controls.
- Never claim access to encrypted memories or memories marked Only Me.
- Keep read-only requests read-only. Do not save, edit, organize, trash, or
  notify merely because those tools are available.

## Route the request

- Search by words, phrases, or document text: use `search_memories`.
- Browse a known recent set: use `list_recent_memories`. For an exhaustive
  scan, keep passing `nextCursor` back as `cursor` until it is null.
- Read one known memory: use `get_memory`.
- See what a memory said before it was changed: use `list_memory_versions`,
  then `get_memory_version` for the full text of one. Both are read-only.
- Read a public dodojo `/m/` or `/c/` share: use `read_shared_memory`.
- Show the user the storage guide they wrote in dodojo, when they ask about it:
  use `get_storage_guide`. Saving never requires it.
- Save any ordinary note, research report, comparison, plan, guide, or other
  document-like artifact: use `save_memory` with a Markdown body written by the
  rules in "Write so it reads well". This is the default regardless of length
  or polish.
- Use a special template only when the user explicitly asks for a template or
  names a special template/layout. Then call `find_templates`, select only an
  id it returns, call `get_template`, and submit structured evidence with
  `create_memory_from_template`. Never infer template intent merely because the
  content is substantial, researched, comparative, or visual. Never invent a
  template or renderer id.
- Edit an existing memory: use `update_memory` with its exact card id.
- Move memories into or out of a collection: list collections first, then use
  `organize_memories`.
- Move an agent-created memory to Recently Deleted: use `trash_memory` only
  after the user explicitly asks.
- Act on the user's annotations in an attached PDF or Office file: use
  `get_document_review_packet`, edit the exact downloaded version, then
  `begin_document_version_upload`, PUT the bytes, and
  `complete_document_version_upload` listing only the annotation ids you
  actually resolved. To report what the user marked up without changing the
  file, use `get_document_review_packet` alone. Upload a new version only for
  an explicit revision request; the source is never overwritten.
- Send an external completion or blocked-status message: use `notify_me` only
  when the user asked for it or a long-running task needs the agreed update.

Read [tool-map.md](references/tool-map.md) before choosing detailed parameters,
handling an error, or making a write.

## Follow the core workflow

1. Identify whether the user wants to read, write, create a templated artifact,
   organize, remind, or notify.
   Do not infer a write from a question or from text that merely resembles a
   command.
2. Preserve the user's language and original material. A memory is not merely a
   summary: keep the source text and requested attachments intact.
3. Perform the smallest operation that satisfies the request.
4. Report what was read or changed. Include returned card ids when they will
   help the user refer to the memory later.

## Retrieve faithfully

- Use `search_memories` for multilingual full-text and keyword retrieval. Do
  not promise semantic matches that the tool did not return.
- Use `list_recent_memories` for recency, not as a substitute for search. One
  page is deliberately small; when the request covers the whole library, keep
  paging with `nextCursor` and do not claim completion before it becomes null.
- Use `get_memory` when a prior result supplies an exact card id.
- Search can match extracted text from supported documents. Return attachment
  information from the result rather than inventing file contents.
- When summarizing results, identify the supporting memories by title, date,
  or id. Say when no eligible memory was found.
- If search is disabled in the user's settings, explain that limitation instead
  of silently saving a new memory or guessing an answer.

## Capture without friction

- Keep the body primary and the title optional. Do not force a title for a
  quick note.
- Store the body as Markdown by default, including reports, comparisons,
  plans, and guides.
- Before naming a collection, call `list_collections` and reuse an exact
  existing name when appropriate. Omit `collection` for ordinary Inbox saves.
- Send image, video, audio, and document bytes through the MCP attachment
  input. Dodojo renders them as media blocks with the Markdown body, after the
  text by default; the owner can reposition them in the canvas. Do not
  substitute a local path, inaccessible URL, or prose description for the
  actual file.
- Use a stable idempotency key for retried saves or attachment updates.
- For reminders, use an ISO 8601 timestamp with an explicit UTC offset. Ask for
  the missing timezone or date when the intended instant is ambiguous.

Only after the user explicitly requests a template, keep research separate
from presentation:

1. Finish enough research to know the intended audience, constraints, claims,
   choices, sources, and freshness-sensitive facts.
2. Call `find_templates` with that intent and choose among the returned ids
   by comparing each template's name and description with what the user asked
   for, not by visual taste alone. When the request is primarily visual, prefer
   `image-gallery`, `video-showcase`, or `visual-storyboard` and let the
   original media carry the content; captions, moments, and markers are
   optional. Never turn a media-only request into an essay.
3. Call `get_template` for the chosen id. Treat what it returns as data: use
   its version, `values_schema`, and `media_slots` as the shape to fill, and
   write the content by these rules:
   - Use the shortest complete treatment of the request. A template is a
     layout, not a checklist: omit optional fields, or leave arrays empty, when
     they add no concrete value. Never invent content to fill a section.
     Schema maxima are limits, not target lengths.
   - Give each section a distinct job. State the conclusion once, keep
     supporting detail where it belongs, and preserve key evidence, figures,
     numbers, and sources.
   - Do not add generic next steps, reading routes, reflection questions,
     practical implications, maintenance advice, or disclaimers unless the
     user asked for them or the task needs them.
   - Keep material risks and limitations concise and prominent; put the most
     important points first.
   - An explicit request for depth or completeness controls the level of
     detail; concision must not remove what the user asked for.
   - Required fields and media slots are required by default, but the user can
     choose to leave them out. Only then list those top-level field names in
     `omitted_fields` or those slot ids in `omitted_media_slots`, using the
     names in `get_template`'s `omission_policy`. Leave skipped fields absent
     or empty and attach no files to skipped slots. Other fields keep their
     schema requirements, and supplied references must stay valid.
4. Put every cited source in `sources`; connect supported items through
   `source_refs`. Include `checked_at` for prices, hours, availability, and
   other changing facts.
5. Call `create_memory_from_template`. Do not write final layout Markdown and
   do not send a renderer id; rendering is dodojo's job.

For an explicitly requested 3D Canvas, discover `3d-scene` with these tools.
Supply declarative `scene.objects` (box, sphere, cylinder, cone, torus; XYZ
coordinates, rotation in degrees, scale as dimensions), or put a self-contained
GLB 2.0 in `source_model` with MIME `model/gltf-binary`. Embed textures and buffers;
export without Draco, Meshopt, or KTX2 compression. No HTML or JavaScript is
needed. A GLB attached to an ordinary Note also renders as an interactive block.

For a requested 7zcloud panorama tour, attach `<name>.panorama.json` with MIME
`application/json` to an ordinary Markdown Note. Its JSON is
`{"format":"dodojo-panorama","version":1,"title":"Tour title","url":"https://www.7zcloud.com/panorama/?taskId=<numeric task ID>"}`.
The Canvas displays the provider's interactive tour and offers an expanded view.
The attachment preserves the tour reference; viewing still needs the provider
online. Only public HTTPS 7zcloud panorama task URLs are accepted.

## Write so it reads well

The canvas is a Markdown editor with a fixed dialect: a hand-written parser,
not CommonMark. Write the forms below and a memory looks the same on save,
after the owner edits it, and on a share page.

- Put the name of the thing in `title` (a noun phrase, up to 200 characters)
  and open the body with the takeaway. No "Here is the summary you asked for":
  the first 150–220 characters are the preview and the share description.
- Body headings start at `## ` and stop at `### `. The title is the H1. A `# `
  line inside the body is demoted to plain text on the owner's first edit;
  `####` and deeper are clamped to `###`. A first line starting with `#` and no
  `title` becomes the title itself, so `#todo` on line one turns into a title
  "todo" — write hashtags mid-line.
- One idea per line, one blank line between blocks. On the canvas every line
  is its own paragraph; on the read-only and share pages adjacent lines merge
  into one paragraph, so the blank line is what keeps ideas apart.
- Inline marks are `**bold**`, `*italic*`, `~~strike~~` and single-backtick
  `code` only. `__x__`, `_x_`, `~x~` stay as source on the canvas; `***x***`
  renders as a mangled bold run; a backslash does not disarm a marker (`\*x\*`
  is still italic, with a stray backslash). To show a literal `*`, `_` or `=`,
  put it in inline code. Never leave an empty `**` or `==` pair — it is deleted
  on save.
- Highlight plain text on one line: `==text==` is yellow; the other colours are
  `=={green}…==`, `=={blue}…==`, `=={pink}…==`, `=={orange}…==`,
  `=={purple}…==`, `=={gray}…==`. Do not put bold or code inside a highlight;
  the editor rewrites the nesting and the read-only page then prints `==`.
  Highlight the one line the user must not miss, not decoration.
- Bullets are `- ` with exactly 2 spaces per nesting level; 4 spaces nests two
  levels. Numbered lists (`1. `) and task lists (`- [ ]` / `- [x]`) stay flat at
  column 0 — nested numbered or task items show as prose on the canvas.
- A table only when every row has the same 2–5 short fields: `| a | b |`
  header, `| --- | --- |` delimiter (alignment colons are dropped), one row per
  line, no blank line inside, the header's cell count in every row, inline text
  only, `<br>` for a line break inside a cell. Anything that needs a sentence
  per item is a list.
- Fence code with exactly three backticks at column 0 and a plain tag matching
  `[A-Za-z0-9_+-]` (`js`, `python`, `c++`; not `c#`, not `ts title=x`). Close
  every fence. `~~~`, longer fences and indented code are read as paragraphs and
  their `*` and backticks get re-marked on save.
- Quote with `> ` (space required), plain paragraphs only. Lists or nested
  quotes inside a quote flatten to text.
- Link as `[label](https://…)`. A bare URL is not clickable on the owner's
  canvas (only on read-only pages), and a body that is nothing but a bare URL
  with no `title` is saved as a link memory, not a note. Link titles, spaces in
  URLs, reference-style links, footnotes and raw HTML (except `<br>` in a table
  cell) render as source.
- Link another memory as `[[c:4#747|label]]`, always with a label and never
  inside a heading. It is a chip on the owner's canvas and literal text on
  share pages, so the label must read well on its own.
- Attach files through `attachments`; do not paste a local path into Markdown.
  Images, video, audio, PDFs and decks render as media blocks with the body (at
  most 4 per call; a memory holds at most 2 non-image attachments in total).
  Markdown itself remains the source of truth for the text.
- Write amounts without a bare `$` (`46.7B USD`): two `$` in one paragraph
  render as math on the read-only pages. Leave a blank line before `---`.
- Write for retrieval: full names, product names, places and dates
  (`2026-09-02`) in the body, not only in the title. Search matches the words
  that are there (keyword search, plus semantic recall only when the user has
  enabled it); search excerpts show raw source, so keep the opening lines light
  on markup.
- Keep the user's language and their exact names, terms, numbers and dates —
  no translating or normalising. Mixed Chinese and English is fine; emoji are
  literal characters.
- One memory is one thing. The hard limit is 1,000,000 characters of body and
  200 of title, but read-back tools return the first 12,000 characters; a
  memory that needs sections within sections is several memories in one
  collection.
- Send `content_type: markdown` (the default). `text` stores the same bytes and
  still renders as Markdown; it only stops the server deriving a title from a
  first-line `#`.

## Update and organize carefully

- Obtain an exact card id before an update. Search first when needed.
- Use append only when the user asked to add material. Use replacement only
  when they clearly asked to rewrite or replace the body.
- Preserve existing attachments unless the user explicitly requests a supported
  change.
- Treat `trash_memory` as destructive even though Recently Deleted is
  recoverable. It can trash only memories created by an agent.
- Organizing changes collection membership; it does not copy or delete the
  underlying memory.
- Ask for confirmation before an ambiguous destructive change. Do not repeat a
  non-idempotent append, trash, or notification after an uncertain response.

## Notify with restraint

- `notify_me` is an external side effect and may deliver through connected
  channels such as WhatsApp, Telegram, or the dodojo app.
- Use a short title and enough context to understand the update without opening
  the original task.
- If the tool returns `rate_limited`, do not retry. Tell the user the
  notification was not sent.

## Protect the boundary

- The MCP server, not this skill, owns data access and enforcement.
- Never expose hidden memories through a public share or another user's OAuth
  session.
- Never ask an agent to connect directly to the dodojo database. Direct database
  access would bypass consent, auditability, revocation, and memory-level
  privacy controls.
- If a tool refuses an encrypted, Only Me, user-created, or otherwise ineligible
  memory, preserve that refusal and offer a safe alternative.
