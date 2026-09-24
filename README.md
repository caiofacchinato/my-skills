# my-skills

Agent Skills pessoais para uso com Claude (Claude Desktop, Claude Code, Cowork). Marcadas com o
sufixo `-42it` pra distinguir das skills padrão/de terceiros em projetos maiores.

## Skills neste repositório

### [`agent-orchestration-plan-42it`](./agent-orchestration-plan-42it)
Cria ou atualiza um plano faseado de prompts para orquestrar agentes de codificação de IA (Claude
Code, Gemini Antigravity, OpenCode etc.) num projeto novo ou existente. Pode gerar só o documento
do plano (para copiar/colar) ou o kit completo de arquivos pronto para o repositório — memória
entre sessões (`CLAUDE.md`, `docs/AUDIT.md`, `docs/DECISIONS.md`, `docs/PROGRESS.md`), um comando
por fase, um comando orquestrador que segue o roadmap sem pular etapas, e subagentes com contexto
isolado para tarefas especializadas.

### [`project-discovery-42it`](./project-discovery-42it)
Explora e documenta o estado atual de um projeto/código já existente, antes de qualquer plano de
fases ser criado — sem propor solução, sem roadmap. Produz `docs/AUDIT.md`, que a skill acima já
sabe ler quando existir, em vez de reexplorar o projeto do zero.

As duas se conectam por esse arquivo: rode `project-discovery-42it` primeiro num projeto já
existente sem mudança específica em mente; rode `agent-orchestration-plan-42it` quando já tiver
algo concreto para planejar.

## Como instalar

**Claude Desktop (upload manual):**
1. Baixe/zipe a pasta da skill que quiser (o conteúdo de `SKILL.md` + `references/` direto na
   raiz do zip, sem pasta extra por cima).
2. No app: `Settings → Capabilities → Skills` (ou `Customize → Skills`) → `Add`/`Create skill` →
   selecione o zip.

**Claude Code (CLI ou aba Code do Desktop):**
```bash
# pessoal, disponível em qualquer projeto:
git clone https://github.com/caiofacchinato/my-skills.git
cp -r my-skills/agent-orchestration-plan-42it ~/.claude/skills/
cp -r my-skills/project-discovery-42it ~/.claude/skills/

# ou só neste projeto:
cp -r my-skills/agent-orchestration-plan-42it <projeto>/.claude/skills/
cp -r my-skills/project-discovery-42it <projeto>/.claude/skills/
```

**Via `skills.sh`** (ferramenta comunitária de instalação entre agentes, não é da Anthropic —
funciona se você já usa esse instalador):
```bash
npx skills add caiofacchinato/my-skills
```

## Licença

MIT — ver [`LICENSE`](./LICENSE). Ajuste se preferir outra.
