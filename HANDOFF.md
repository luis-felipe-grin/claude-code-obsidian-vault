# HANDOFF — Boson Agents / UGC Factory
**Data:** 2026-05-19  
**Sprint atual:** Sprint 01 — Setup Completo (semana 1)

---

## PASSO 0 — Cole este prompt no Claude ao abrir (copie tudo abaixo)

```
Leia os seguintes arquivos nesta ordem e confirme que leu cada um:

1. f:\Claude Code + Obsidian\CLAUDE.md
2. f:\Claude Code + Obsidian\Agência de IA\CLAUDE.md
3. f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Sprint-01.md
4. f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Backlog.md

Após ler, me diga:
- Qual é o projeto
- Quais são as tasks da Sprint 01
- Quais são as pendências técnicas no backlog

Estou no PC novo. Vamos começar a Sprint 01.
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

## PASSO 2 — Resultado esperado nas árvores

### VS Code (raiz: `f:\Claude Code + Obsidian\`)
```
CLAUDE CODE + OBSIDIAN
├── .claude\
├── Agência de IA\                ← vault Obsidian
│   ├── .claude\
│   ├── .obsidian\
│   ├── ⚙️ Operações\
│   │   ├── templates\
│   │   ├── SOP-criar-nova-influencer.md
│   │   └── VAULT-SETUP-PROMPT.md
│   ├── 🌍 Mercados\
│   ├── 🎭 Influencers\
│   ├── 👤 Pessoas\
│   ├── 👥 Clientes\
│   ├── 💼 Projetos\
│   ├── 📁 Arquivo\
│   ├── 📅 Diário\
│   ├── 📅 Scrum\
│   │   ├── Backlog.md
│   │   ├── Epics.md
│   │   ├── README.md
│   │   ├── Sprint-01.md   ← sprint atual
│   │   ├── Sprint-02.md
│   │   ├── Sprint-03.md
│   │   ├── Sprint-04.md
│   │   └── Times.md
│   ├── 📊 Métricas\
│   ├── 📊 Reuniões\
│   ├── 📚 Pesquisa\
│   │   └── DNA-Viral.md
│   ├── 📣 Conteúdo\
│   ├── 📥 Inbox\
│   ├── 🤝 Deals\
│   ├── 🛍️ Produtos\
│   ├── Clippings\         ← 24 clippings do curso Phase 1 e 2
│   ├── .gitignore
│   ├── Bem-vindo.md
│   └── CLAUDE.md          ← contexto principal
├── .gitignore
├── CLAUDE.md              ← ponto de entrada para o Claude
├── DEVLOG.md
├── HANDOFF.md             ← este arquivo
├── HANDOFF-OBSIDIAN-SETUP.md
└── videoplayback.txt
```

### Obsidian (vault: `f:\Claude Code + Obsidian\Agência de IA\`)
```
Agência de IA
├── ⚙️ Operações
├── 🌍 Mercados
├── 🎭 Influencers
├── 👤 Pessoas
├── 👥 Clientes
├── 💼 Projetos
├── 📁 Arquivo
├── 📅 Diário
├── 📅 Scrum
│   ├── Backlog
│   ├── Epics
│   ├── README
│   ├── Sprint-01   ← sprint atual
│   ├── Sprint-02
│   ├── Sprint-03
│   ├── Sprint-04
│   └── Times
├── 📊 Métricas
├── 📊 Reuniões
├── 📚 Pesquisa
│   └── DNA-Viral
├── 📣 Conteúdo
├── 📥 Inbox
├── 🤝 Deals
├── 🛍️ Produtos
├── Clippings
├── Bem-vindo
└── CLAUDE
```

---

## PASSO 3 — Sincronizar (quando vindo do PC original)

```powershell
cd "f:\Claude Code + Obsidian"                   ; git pull origin master
cd "f:\Claude Code + Obsidian\Agência de IA"     ; git pull origin master
cd "f:\ugc-factory"                              ; git pull origin main
cd "f:\influenceros"                             ; git pull
cd "f:\content-creator"                          ; git pull
```

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
| DNA Viral Agent — `POST /analyze-product` | Alta |
| Bug Kontext — `genMode="kontext"` chama PuLID | Média |
| Vitrine PDF → `🛍️ Produtos/` por mercado | Baixa |
| Skill `create_influencer_identity()` | Futura |

**Arquivo completo:** `f:\Claude Code + Obsidian\Agência de IA\📅 Scrum\Backlog.md`

---

## Contexto Rápido do Projeto

**Boson Agents** = agência de influencers de IA para TikTok Shop em 5 mercados: BR, UK, DE, FR, US.

**Regras críticas:**
1. Git obrigatório — qualquer pasta nova: `git init` + `gh repo create` antes de qualquer trabalho
2. Claude gera roteiro — NUNCA ChatGPT em nenhuma etapa de texto
3. Scrum é fonte de verdade — sempre consultar `Sprint-XX.md` antes de sugerir o que fazer
4. Tudo = skill candidata — avaliar `src/skills/` antes de criar código

**Repos GitHub (conta: luis-felipe-grin):**
- `claude-code-obsidian-vault` → `f:\Claude Code + Obsidian`
- `boson-agents-vault` → `f:\Claude Code + Obsidian\Agência de IA`
- `ugc-factory` → `f:\ugc-factory`
- `influenceros` → `f:\influenceros`
- `content-creator` → `f:\content-creator`
