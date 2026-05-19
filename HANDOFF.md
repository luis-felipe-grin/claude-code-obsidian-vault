# HANDOFF — Boson Agents / UGC Factory
**Última atualização:** 2026-05-19  
**Sprint atual:** Sprint 01 — Setup Completo (semana 1)

---

## PASSO 0 — Cole este prompt no Claude ao abrir (copie tudo abaixo)

```
Leia os seguintes arquivos nesta ordem e confirme que leu cada um:

1. f:\Claude Code + Obsidian\CLAUDE.md
2. f:\Claude Code + Obsidian\Agência de IA\CLAUDE.md
3. f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Sprint-01.md
4. f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Times.md
5. f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Backlog.md

Após ler, me diga:
- Qual é o projeto e quais times existem
- Quais são as tasks da Sprint 01
- Quais são as pendências técnicas no backlog

Estou continuando a Sprint 01. Vamos trabalhar.
```

---

## PASSO 1 — Setup no PC novo

### 1.1 Clonar os repos (PowerShell como Admin)

```powershell
git clone https://github.com/luis-felipe-grin/claude-code-obsidian-vault "f:\Claude Code + Obsidian"
git clone https://github.com/luis-felipe-grin/boson-agents-vault "f:\Claude Code + Obsidian\Agência de IA"
git clone https://github.com/luis-felipe-grin/ugc-factory "f:\ugc-factory"
git clone https://github.com/luis-felipe-grin/influenceros "f:\influenceros"
git clone https://github.com/luis-felipe-grin/content-creator "f:\content-creator"
```

### 1.2 Abrir no VS Code

```powershell
code "f:\Claude Code + Obsidian"
```

### 1.3 Abrir no Obsidian

1. Obsidian → **"Open folder as vault"**
2. Selecionar: `f:\Claude Code + Obsidian\Agência de IA\`

> Abrir `Agência de IA\` — não a pasta pai.

### 1.4 Abrir o Claude Code

No terminal do VS Code (já na pasta `f:\Claude Code + Obsidian\`):
```bash
claude
```

Cole o prompt do **PASSO 0** para dar contexto completo ao Claude.

---

## PASSO 2 — Sincronizar (quando vindo do PC original)

```powershell
cd "f:\Claude Code + Obsidian"                   ; git pull origin master
cd "f:\Claude Code + Obsidian\Agência de IA"     ; git pull origin master
cd "f:\ugc-factory"                              ; git pull origin main
cd "f:\influenceros"                             ; git pull
cd "f:\content-creator"                          ; git pull
```

---

## O que foi feito até aqui (sessão 2026-05-19)

### Skills Python — `f:\ugc-factory\src\skills\`

8 novos arquivos de skill criados (stubs com NotImplementedError, aguardando clippings do curso):

| Arquivo | Phase | Prioridade |
|---------|-------|-----------|
| `animate.py` | Phase 3 | Normal |
| `add_audio.py` | Phase 4 | Normal |
| `edit_video.py` | Phase 4 | Normal |
| `create_influencer_identity.py` | Phase 5 | Normal |
| `generate_ugc_video.py` | **Phase 6** | ⚡ PRIORITÁRIA |
| `generate_caption.py` | Phase 7 | Normal |
| `generate_script.py` | Phase 8 | Normal |
| `clone_influencer.py` | **Phase 9** | ⚡ PRIORITÁRIA |

`__init__.py` atualizado com mapa completo Phase→Skill.

### Times PM — `f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Times.md`

13 agentes PM no organograma (4 existentes + 9 novos):

**Pesquisa de Produto:**
- `Claude-DNA` — análise DNA Viral (produto → hooks + persona + CTAs)
- `Claude-Copy` — copywriting Módulo 5 (hooks 3s, CTAs invisíveis)

**Produção AI:**
- `Claude-Video` — Phase 3 animação
- `Claude-Edit` — Phase 4 edição + áudio
- `Claude-Identity` — Phase 5 identidade de influencer
- `Claude-UGC` ⚡ — Phase 6 pipeline UGC ponta-a-ponta

**Conteúdo & Social:**
- `Claude-Script` — Phase 8 roteiros (NUNCA ChatGPT)
- `Claude-Social` — Phase 7 conteúdo social + calendário

**Escala:**
- `Claude-Clone` ⚡ — Phase 9 clone digital + automação 24/7

### InfluencerOS Sidebar — `f:\influenceros\`

3 novas seções adicionadas + 8 páginas placeholder criadas:

| Seção | Rota | Time |
|-------|------|------|
| Pesquisa de Produto | `/dna-viral` | Claude-DNA |
| Pesquisa de Produto | `/copywriting` | Claude-Copy |
| Pesquisa de Produto | `/vitrine` | Claude-Product |
| Produção AI | `/scripts` | Claude-Script |
| Produção AI | `/videos` | Claude-Video + Claude-UGC |
| Produção AI | `/identidades` | Claude-Identity |
| Escala | `/social` | Claude-Social |
| Escala | `/clone` | Claude-Clone |

### Fichas de Produto — `f:\Claude Code + Obsidian\Agência de IA\🛍️ Produtos\`

20 fichas criadas (DNA Viral ainda vazio — rodar `[Claude-DNA]` em cada uma):

- **🇬🇧 UK (7):** `adjustable-posture-corrector.md` ⭐31k vendas PRIORIDADE, dr-dent, arabiyat, halara, wellgard, womens-quick-dry, womens-deep
- **🇩🇪 DE (6):** `organizador-geladeira.md` ⭐50k vendas PRIORIDADE, vegetable-chopper 38k, faixa-modeladora 19k, magnesium-complex, peel-stick-marble, denim-v-neck
- **🇫🇷 FR (6):** collagene-marin, windboss-gommes-shilajit, complexe-magnesium-nutribrain, nuclever-cortisol-manager, plus-the-cheek, mens-watch-perfume-set
- **🇧🇷 BR (1):** kit-2-calca-legging

Template base: `⚙️ Operações/templates/ficha-produto.md`

---

## Sprint 01 — Tasks (referência rápida)

**Meta:** 2 contas TikTok ativas e aquecidas + produto escolhido

| Task | Dono | Status |
|------|------|--------|
| Configurar VPN TunnelBear | Felipe | ⬜ |
| Criar conta TikTok UK | Felipe | ⬜ |
| Criar conta TikTok DE | Felipe | ⬜ |
| Aquecer contas (Game Filter — 100 views/24h) | Felipe | ⬜ |
| Acessar vitrine de produtos virais (Jéssica) | Felipe | ⬜ |
| Escolher produto para UK | Felipe+Claude | ⬜ |
| Escolher produto para DE | Felipe+Claude | ⬜ |

**Arquivo completo:** `f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Sprint-01.md`

---

## Pendências Técnicas (Backlog)

| Task | Prioridade |
|------|-----------|
| DNA Viral Agent — `dna_viral.py` + `POST /analyze-product` + tab "/dna-viral" no InfluencerOS | **Alta** |
| Bug Kontext — `genMode="kontext"` chama PuLID em vez de `generate_image_kontext()` | Média |
| Preencher DNA Viral das 20 fichas de produto com `[Claude-DNA]` | Média |
| Implementar skills Phase 6 + 9 (`generate_ugc_video.py`, `clone_influencer.py`) | Alta |
| Implementar skills Phase 3-5, 7-8 (stubs aguardando clippings) | Baixa |

**Arquivo completo:** `f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Backlog.md`

---

## Contexto Rápido do Projeto

**Boson Agents** = agência de influencers de IA para TikTok Shop em 5 mercados: BR, UK, DE, FR, US.

**Regras críticas:**
1. Git obrigatório — qualquer pasta nova: `git init` + `gh repo create` antes de qualquer trabalho
2. Claude gera roteiro — NUNCA ChatGPT em nenhuma etapa de texto
3. Scrum é fonte de verdade — sempre consultar `Sprint-XX.md` antes de sugerir o que fazer
4. Tudo = skill candidata — avaliar `src/skills/` antes de criar código
5. Phases 6 e 9 são PRIORITÁRIAS — core do negócio Boson Agents

**Repos GitHub (conta: luis-felipe-grin):**
- `claude-code-obsidian-vault` → `f:\Claude Code + Obsidian`
- `boson-agents-vault` → `f:\Claude Code + Obsidian\Agência de IA`
- `ugc-factory` → `f:\ugc-factory`
- `influenceros` → `f:\influenceros`
- `content-creator` → `f:\content-creator`
