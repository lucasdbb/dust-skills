---
name: dust-breaking-changes
dust_skill_id: skl_bnZYnNUMhHW8
availability: workspace_users
---

# dust-breaking-changes

**Description:** Checklist and process for shipping a breaking change safely in the Dust monorepo (API contracts, DB schema, SDK types). Use when a change could break existing clients, extensions, or running workflows.

## Instructions

(Condensed — fetch full instructions from Dust via `building_agents_and_skills__describe_skill` id `skl_bnZYnNUMhHW8`. This skill walks through: identifying who depends on the changing surface (extension, public API clients, SDK consumers, internal workers), the expand/contract pattern for DB and API changes, versioning the public API when needed, coordinating deploy order between `core`, `front`, `connectors`, and feature-flagging risky changes before full rollout.)
