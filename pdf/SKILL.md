---
name: pdf
dust_skill_id: skl_se8GjlHmCYyf
availability: workspace_users
note: Matches Anthropic's publicly published "pdf" skill (github.com/anthropics/skills). Full source not reproduced here — pull it from the public repo for verbatim content (pypdf/pdfplumber/reportlab scripts, form filling, OCR via Tesseract, etc.).
---

# pdf

**Description:** Use this skill whenever the user wants to do anything with PDF files. This includes reading or extracting text/tables from PDFs, combining or merging multiple PDFs into one, splitting PDFs apart, rotating pages, adding watermarks, creating new PDFs, filling PDF forms, encrypting/decrypting PDFs, extracting images, and OCR on scanned PDFs to make them searchable. If the user mentions a .pdf file or asks to produce one, use this skill.

## Instructions (summary)
Covers, with Python (pypdf, pdfplumber, reportlab) and command-line tools (pdftk, qpdf, poppler-utils, Tesseract OCR):
- Text/table extraction (including from scanned PDFs via OCR)
- Merge/split/rotate/watermark operations
- Creating new PDFs from scratch (reportlab)
- Filling existing PDF forms (field discovery, then fill)
- Encrypting/decrypting and permission management
- Image extraction from PDF pages
- Validation after any edit (page count, text sanity check)

Get the full verbatim instructions and bundled scripts from the public Anthropic skills repository or by re-running `building_agents_and_skills__describe_skill` with id `skl_se8GjlHmCYyf` in Dust.
