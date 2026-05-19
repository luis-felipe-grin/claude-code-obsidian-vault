# Handoff — Reproduzir ambiente exato em outro PC

> Objetivo: chegar exatamente nesta árvore no VS Code + Obsidian apontando para as pastas certas.

---

## Resultado esperado

**VS Code** abre em `f:\Claude Code + Obsidian\` e mostra:
```
CLAUDE CODE + OBSIDIAN          ← raiz do workspace (VS Code abre AQUI)
├── .claude\
├── Agência de IA\              ← vault do Obsidian (Obsidian abre AQUI)
│   ├── .claude\
│   ├── .obsidian\
│   ├── ⚙️ Operações\
│   │   └── templates\
│   ├── 🌍 Mercados\
│   ├── 🎭 Influencers\
│   ├── 👤 Pessoas\
│   ├── 👥 Clientes\
│   ├── 💼 Projetos\
│   ├── 📁 Arquivo\
│   ├── 📅 Diário\
│   ├── 📅 Scrum\
│   ├── 📊 Métricas\
│   ├── 📊 Reuniões\
│   ├── 📚 Pesquisa\
│   ├── 📣 Conteúdo\
│   ├── 📥 Inbox\
│   ├── 🤝 Deals\
│   ├── 🛍️ Produtos\
│   ├── Clippings\
│   ├── .gitignore
│   ├── Bem-vindo.md
│   └── CLAUDE.md               ← contexto principal — Claude lê primeiro
├── .gitignore
├── DEVLOG.md                   ← histórico completo do projeto
├── HANDOFF-OBSIDIAN-SETUP.md   ← este arquivo
└── videoplayback.txt
```

---

## Passo a Passo

### 1. Clonar os dois repos

```bash
# Repo externo (DEVLOG + workspace raiz)
git clone https://github.com/luis-felipe-grin/claude-code-obsidian-vault "f:\Claude Code + Obsidian"

# Vault Obsidian (dentro do repo externo)
git clone https://github.com/luis-felipe-grin/boson-agents-vault "f:\Claude Code + Obsidian\Agência de IA"
```

> As pastas com emoji (⚙️ Operações, 📅 Scrum, etc.) já vêm corretas do git — não recriar manualmente.

---

### 2. Abrir no VS Code

```bash
code "f:\Claude Code + Obsidian"
```

VS Code deve mostrar a árvore completa com `Agência de IA\` dentro, incluindo as subpastas com emoji.

---

### 3. Abrir no Obsidian

1. Abrir Obsidian
2. **"Open folder as vault"**
3. Selecionar: `f:\Claude Code + Obsidian\Agência de IA\`

> **Não** abrir `f:\Claude Code + Obsidian\` como vault — o Obsidian deve apontar para a subpasta `Agência de IA\`.

---

### 4. Abrir o Claude Code

No terminal dentro do VS Code (já na pasta `f:\Claude Code + Obsidian\`):

```bash
claude
```

O Claude vai ler automaticamente `Agência de IA\CLAUDE.md` e ter contexto completo do projeto.

---

### 5. Clonar os outros repos do ecossistema (opcional mas recomendado)

```bash
git clone https://github.com/luis-felipe-grin/content-creator "f:\content-creator"
git clone https://github.com/luis-felipe-grin/ugc-factory "f:\ugc-factory"
git clone https://github.com/luis-felipe-grin/influenceros "f:\influenceros"
```

---

## Sincronizar (quando voltar ao PC original)

```bash
cd "f:\Claude Code + Obsidian"          && git pull origin master
cd "f:\Claude Code + Obsidian\Agência de IA" && git pull origin master
cd "f:\content-creator"                 && git pull origin master
cd "f:\ugc-factory"                     && git pull origin main
cd "f:\influenceros"                    && git pull
```

---

## Importante

- As pastas com emoji no nome são intencionais — manter exatamente como estão
- `.obsidian\` controla as configurações do Obsidian (tema, plugins, graph) — nunca apagar
- `.claude\` contém instruções específicas para o Claude — nunca apagar
- `CLAUDE.md` dentro do vault é o contexto principal — o Claude sempre lê este arquivo primeiro
- `DEVLOG.md` na raiz tem o histórico completo de tudo que foi feito no projeto
