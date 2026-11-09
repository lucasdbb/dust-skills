---
name: xlsx
dust_skill_id: skl_LKpnHcYFbDJk
availability: workspace_users
note: Matches Anthropic's publicly published "xlsx" skill. Full source not reproduced here.
---

# xlsx

**Description:** Use this skill any time a spreadsheet file is the primary input or output: opening, reading, editing, or fixing an existing .xlsx, .xlsm, .csv, or .tsv file (adding columns, computing formulas, formatting, charting, cleaning messy data); creating a new spreadsheet from scratch or from other data sources; or converting between tabular file formats. Also triggers for cleaning/restructuring messy tabular data. Do NOT trigger when the primary deliverable is a Word document, HTML report, standalone Python script, database pipeline, or Google Sheets API integration.

## Instructions (summary)
Covers reading/writing workbooks with openpyxl/pandas, preserving formulas and cached values, formatting (number formats, conditional formatting, charts), and an `xlsx_inspect`-style paginated structural inspection pattern so existing templates are adapted in place rather than rewritten from scratch.

Get the full verbatim instructions and bundled scripts from the public Anthropic skills repository or by re-running `building_agents_and_skills__describe_skill` with id `skl_LKpnHcYFbDJk` in Dust.
