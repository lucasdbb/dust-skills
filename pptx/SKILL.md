---
name: pptx
dust_skill_id: skl_GpUUclJm9kgY
availability: workspace_users
note: Matches Anthropic's publicly published "pptx" skill. Full source not reproduced here.
---

# pptx

**Description:** Use this skill any time a .pptx file is involved in any way — as input, output, or both. This includes: creating slide decks, pitch decks, or presentations; reading, parsing, or extracting text from any .pptx file; editing, modifying, or updating existing presentations; combining or splitting slide files; working with templates, layouts, speaker notes, or comments. Trigger whenever the user mentions "deck," "slides," "presentation," or references a .pptx filename, regardless of what they plan to do with the content afterward.

## Instructions (summary)
Same ZIP/XML-based approach as the `docx` skill: unpack/inspect/pack scripts for editing an existing deck's raw XML (layouts, shapes, speaker notes, comments), plus `pptxgenjs`-based generation for building a new deck from scratch, with explicit guidance to adapt existing decks in place with `python-pptx`/XML editing rather than rebuilding from scratch when a source deck already exists.

Get the full verbatim instructions and bundled scripts from the public Anthropic skills repository or by re-running `building_agents_and_skills__describe_skill` with id `skl_GpUUclJm9kgY` in Dust.
