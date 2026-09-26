# Referência — adaptando o kit para outras ferramentas agentic

O kit descrito em `claude_code_kit.md` funciona por **contexto**: um `CLAUDE.md` com o Protocolo de
fases, arquivos de memória, e um arquivo markdown por fase em `docs/fases/`. Isso generaliza para
qualquer ferramenta agentic que leia um arquivo de instruções do repositório. O que é específico do
Claude Code é só a parte de subagentes (`.claude/agents/`) e os comandos nomeados opcionais
(`.claude/commands/`). Adapte por partes, não tudo ou nada, e não invente uma convenção sem
confirmar.

## O que é universal — funciona em qualquer ferramenta agentic decente

- `CLAUDE.md` — a maioria das ferramentas aceita `AGENTS.md` como equivalente (é praticamente um
  padrão informal hoje, adotado por várias ferramentas além do Claude Code). Gere os dois com o
  mesmo conteúdo, ou gere só `AGENTS.md` se o usuário não usa Claude Code. **A seção "Protocolo de
  fases" mora aqui** — é ela que permite dizer só "próximo" ou "começar fase-1c" em qualquer
  ferramenta que carregue esse arquivo em toda sessão.
- `docs/AUDIT.md`, `docs/DECISIONS.md`, `docs/PROGRESS.md` — são arquivos de memória lidos pelo
  próprio agente via instrução no prompt, não dependem de nenhuma convenção específica de
  ferramenta.
- `docs/fases/<fase>.md` — um arquivo markdown por fase, exatamente o mesmo conteúdo do kit do
  Claude Code (já é markdown simples, sem front matter específico). Resolve o problema de esquecer
  uma fase no meio de muitas, e é lido pelo agente só quando o Protocolo de fases resolve aquela
  fase.

Se a ferramenta não carrega `AGENTS.md`/`CLAUDE.md` sozinha em toda sessão, o protocolo não dispara
por conta própria: confirme na documentação dela qual arquivo de instruções persistentes ela lê, ou
avise o usuário que ele precisará colar o Protocolo de fases no início de cada sessão.

## Comandos nomeados (opcional, só como atalho)

Não são necessários — o Protocolo de fases já cobre o mesmo uso por linguagem natural. Só gere se o
usuário pedir um atalho tipo `/fase-1c`. Verifique se a ferramenta escolhida tem esse recurso e onde
ela espera os arquivos antes de gerar qualquer coisa nesse formato — isso muda com frequência entre
versões das ferramentas, então confirme na documentação atual em vez de assumir. Pelo que se sabe
até este momento, como ponto de partida (não como garantia — pesquise antes de gerar):

- **Gemini Antigravity**: prompts pessoais em `~/.antigravity/prompts/` (ou o caminho equivalente
  do editor no Windows/Linux).
- **OpenCode**: comandos em `~/.config/opencode/commands/`.

Qualquer comando gerado deve ser um arquivo fino que aponta para `docs/fases/<fase>.md` e para o
Protocolo de fases, nunca uma cópia do prompt.

## Subagentes

Isso é o que menos generaliza. Os subagentes com contexto isolado e restrição de `tools` são um
recurso documentado do Claude Code hoje; outras ferramentas podem ter algo parecido sob outro nome,
ou nada equivalente, e contexto sozinho não substitui isso. Não invente uma sintaxe sem confirmar na
documentação da ferramenta — é melhor entregar só o kit por contexto e avisar que os subagentes, por
enquanto, são a parte pensada especificamente para o Claude Code.

## Na prática

Se o usuário disser qual ferramenta vai usar e não for Claude Code:

1. Gere sempre o kit "universal" da primeira seção acima (`AGENTS.md` com o Protocolo de fases,
   arquivos de memória, `docs/fases/`).
2. Confirme qual arquivo de instruções persistentes a ferramenta lê antes de assumir `AGENTS.md`.
3. Só se o usuário pedir atalhos nomeados, pesquise rapidamente a convenção de comando dessa
   ferramenta antes de gerar arquivo nela — não chute o caminho nem o formato de front matter.
4. Seja direto sobre o que não dá para replicar (subagentes com contexto isolado) e ofereça a
   alternativa: o kit por contexto já entrega o essencial — "próximo" e "começar fase-X" funcionando
   — mesmo sem essa parte.
