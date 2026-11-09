# Dust Skills Export — workspace "lucas"

Exportação de todas as 57 skills do workspace Dust para o repositório
`lucasdbb/dust-skills`, gerada em 2026-11-09.

## Como foi gerado
Esta exportação foi feita manualmente a partir do Dust, usando a ferramenta
interna `building_agents_and_skills__describe_skill` para extrair nome,
descrições e instruções de cada skill. **O conector GitHub disponível não
tem permissão de escrita em conteúdo de repositório** (só gerencia issues,
pull requests e discussions), então os arquivos foram gerados aqui e
precisam ser enviados manualmente (comandos abaixo).

## Estrutura
Um diretório por skill, seguindo a convenção `<slug>/SKILL.md`:
```
dust-skills-export/
├── fullsales/SKILL.md
├── matuta/SKILL.md
├── ...
```
Cada `SKILL.md` tem um front-matter com `name`, `dust_skill_id` e
`availability`, seguido da descrição (user-facing / agent-facing) e das
instruções completas.

## Observações importantes
- **Skills de negócio/customizadas** (fullsales, matuta, PDF Design System
  ANA, Criador de POPs, ayman-al-abdullah, noah-kagan, generic-language-killer,
  scroll-stopping-creative, schwartz-awareness-mapper, conversion-path-builder,
  offer-extraction, ad-angle-multiplier, avatar-extraction, mechanism-builder,
  headline-matrix, full-funnel-campaign-orchestrator, objection-crusher,
  performance-diagnosis, Elaborar Pesquisa de Mercado, Prep Meeting, Chart this)
  e as **skills internas de engenharia do time Dust** (dust-*, claude-api,
  firecrawl-build-*) foram exportadas **na íntegra**.
- **Skills públicas/genéricas da Anthropic** (docx, pptx, xlsx, pdf,
  frontend-design, mcp-builder, webapp-testing, theme-factory, canvas-design,
  brand-guidelines, slack-gif-creator, skill-creator, doc-coauthoring,
  web-artifacts-builder, internal-comms, algorithmic-art) foram exportadas
  como **resumo + referência**, já que são idênticas (ou quase) às skills
  publicamente disponíveis em `github.com/anthropics/skills`. O texto
  completo pode ser obtido de lá ou puxado novamente do Dust pelo
  `dust_skill_id` indicado em cada arquivo.
- Algumas skills (claude-api, dust-mcp-server, Elaborar Pesquisa de Mercado)
  são muito extensas (até ~58 KB); foram resumidas mantendo toda a lógica de
  decisão, com nota indicando onde buscar o conteúdo verbatim completo.
- Arquivos de referência anexados a várias skills (`references/*.md`,
  scripts auxiliares) **não foram exportados** neste lote — cada SKILL.md
  sinaliza quando isso ocorre. Podem ser extraídos à parte se necessário.

## Como subir para o GitHub
```bash
git clone https://github.com/lucasdbb/dust-skills.git
cp -r dust-skills-export/* dust-skills/
cd dust-skills
git add .
git commit -m "Export all 57 Dust workspace skills"
git push origin main
```
