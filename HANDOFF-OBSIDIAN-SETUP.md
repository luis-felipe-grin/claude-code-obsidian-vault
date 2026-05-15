# Handoff — Setup Obsidian + Claude Code

> Cole este arquivo inteiro no Claude Code do outro PC para replicar a configuração.

---

## Contexto (do vídeo transcrito)

O framework é: **Obsidian como segundo cérebro + Claude Code como copiloto** — os dois abertos na mesma pasta. O Claude lê, escreve e organiza os arquivos markdown do vault automaticamente.

Referência: vídeo de um criador brasileiro mostrando o framework do Andrej Karpathy (ex-Tesla/OpenAI) de usar LLM + wiki local.

---

## Extensão Chrome necessária

**Obsidian Web Clipper** — instale no Chrome para salvar artigos, páginas e conteúdo diretamente no vault sem copiar e colar.
- Buscar na Chrome Web Store por: `Obsidian Web Clipper`
- Após instalar: configurar o vault de destino nas configurações da extensão

---

## Passo a Passo de Instalação

1. Baixar Obsidian em obsidian.md (grátis)
2. Criar um vault novo — **anotar o caminho da pasta**
3. Abrir Claude Code (VS Code ou terminal) **na mesma pasta do vault**
4. Colar o prompt de setup abaixo no Claude Code
5. Responder as perguntas e confirmar a criação

---

## Prompt de Setup do Vault

Cole no Claude Code:

```
Você é meu assistente estratégico e operacional. Vamos configurar meu segundo cérebro no Obsidian para a Agência de IA.

Antes de criar qualquer coisa, me responda estas perguntas para personalizar o vault:

1. Qual é o nome da agência?
2. Quais são os 3 principais serviços que você oferece hoje?
3. Quantos clientes ativos você tem agora? (pode ser uma estimativa)
4. Qual é sua maior dor operacional hoje? (ex: perder follow-up de deals, projetos sem documentação, falta de processo de onboarding)
5. Você quer incluir gestão de conteúdo/marketing no vault? (sim/não)
6. Você tem sócios ou equipe? Quantas pessoas?

Após minhas respostas, crie a seguinte estrutura no vault atual:

ESTRUTURA DE PASTAS:
- 📥 Inbox/
- 📅 Diário/
- 👥 Clientes/
- 💼 Projetos/
- 🤝 Deals/
- 👤 Pessoas/
- 📊 Reuniões/
- 📚 Pesquisa/
- 📣 Conteúdo/
- ⚙️ Operações/
- ⚙️ Operações/templates/
- 📁 Arquivo/

ARQUIVOS A CRIAR:
1. CLAUDE.md na raiz — contexto completo da agência para que Claude nunca precise perguntar quem você é
2. ⚙️ Operações/templates/cliente.md — template padrão para novo cliente
3. ⚙️ Operações/templates/projeto.md — template padrão para novo projeto
4. ⚙️ Operações/templates/reuniao.md — template padrão para reuniões
5. ⚙️ Operações/templates/deal.md — template padrão para pipeline comercial
6. ⚙️ Operações/templates/roteiro-video.md — template para roteiros UGC
7. ⚙️ Operações/templates/influencer-persona.md — template para perfil de influencer IA
8. 📅 Diário/{{data-hoje}}.md — primeiro diário com contexto do setup de hoje

CLAUDE.md deve conter:
- Nome e descrição da agência
- Serviços oferecidos (influencers IA, UGC, afiliados)
- Tom de comunicação preferido
- Estrutura do vault explicada
- Como Claude deve agir em cada situação (reunião, deal, projeto, pesquisa)
- Status de deals: 🔴 Frio | 🟡 Morno | 🟢 Quente | ✅ Fechado | ❌ Perdido
- Status de projetos: 📋 Planejamento | 🔄 Em Andamento | ⏸️ Pausado | ✅ Concluído
- Referência ao sistema InfluencerOS (Next.js) em desenvolvimento

Após criar tudo, pergunte se quero instalar as configurações globalmente ou só neste vault.

Confirme a estrutura antes de criar. Mostre um preview resumido e aguarde meu "pode criar".
```

---

## Prompts do Dia a Dia (após setup)

### Após reunião:
```
Acabei de ter uma reunião com [Nome/Empresa]. Transcrição: [colar]
Crie a nota em 📊 Reuniões/, atualize o cliente em 👥 Clientes/ com próximos passos, e se houver deal, atualize em 🤝 Deals/.
```

### Pipeline comercial:
```
Analise todos os deals em 🤝 Deals/ e me diga: quais são mais propensos a fechar, onde devo focar essa semana, e o que está parado há mais de 7 dias.
```

### Preparação para reunião:
```
Tenho reunião com [Nome] amanhã. Leia o arquivo em 👥 Clientes/, as últimas reuniões em 📊 Reuniões/ e o deal em 🤝 Deals/. Me prepare: contexto, pendências e 3 perguntas estratégicas.
```

### Salvar artigo/conteúdo:
```
Organizei esse artigo no inbox: [nome]. Mova para 📚 Pesquisa/, crie resumo de 3 bullet points e diga se tem aplicação para algum cliente ou projeto atual.
```

---

## Vault já existente neste PC

Vault configurado em: `F:\Claude Code + Obsidian\Agência de IA\`

Arquivos já criados:
- `CLAUDE.md` — contexto da agência
- `⚙️ Operações/VAULT-SETUP-PROMPT.md` — este mesmo prompt
- Templates: cliente, projeto, reunião, deal, roteiro-video, influencer-persona
- `🌍 Mercados/` — brasil, estados-unidos, reino-unido, alemanha, franca
- `📅 Diário/2026-05-04.md`

Para sincronizar entre PCs: usar Git ou copiar a pasta manualmente.
