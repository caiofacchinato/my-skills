# Referência — adaptando o kit para outras ferramentas agentic

O kit completo (comandos de fase + orquestrador + subagentes) descrito em
`claude_code_kit.md` usa convenções específicas do Claude Code (`.claude/commands/`,
`.claude/agents/`). Nem toda ferramenta tem as três peças — comando nomeado, subagente
configurável, orquestração por arquivo — então adapte por partes, não tudo ou nada, e não invente
uma convenção sem confirmar.

## O que é universal — funciona em qualquer ferramenta agentic decente

- `CLAUDE.md` — a maioria das ferramentas aceita `AGENTS.md` como equivalente (é praticamente um
  padrão informal hoje, adotado por várias ferramentas além do Claude Code). Gere os dois com o
  mesmo conteúdo, ou gere só `AGENTS.md` se o usuário não usa Claude Code.
- `docs/AUDIT.md`, `docs/DECISIONS.md`, `docs/PROGRESS.md` — são arquivos de memória lidos pelo
  próprio agente via instrução no prompt, não dependem de nenhuma convenção específica de
  ferramenta.
- Um arquivo markdown por fase (o mesmo conteúdo de `.claude/commands/<fase>.md`, sem o
  front matter específico do Claude Code) — sempre útil, mesmo que a ferramenta não tenha "comando
  nomeado": já fica organizado em arquivos separados em vez de um prompt gigante, o que resolve o
  problema de esquecer uma fase no meio de muitas, mesmo sem automação nenhuma por cima.

## Comandos nomeados (equivalente a `/fase-0`)

Verifique se a ferramenta escolhida tem esse recurso e onde ela espera os arquivos antes de gerar
qualquer coisa nesse formato — isso muda com frequência entre versões das ferramentas, então
confirme na documentação atual em vez de assumir. Pelo que se sabe até este momento, como ponto de
partida (não como garantia — pesquise antes de gerar):

- **Gemini Antigravity**: prompts pessoais em `~/.antigravity/prompts/` (ou o caminho equivalente
  do editor no Windows/Linux).
- **OpenCode**: comandos em `~/.config/opencode/commands/`.

## Subagentes e orquestração automática

Isso é o que menos generaliza. O orquestrador (`proximo.md`) e os subagentes com contexto isolado
e restrição de `tools` são um recurso documentado do Claude Code hoje; outras ferramentas podem
ter algo parecido sob outro nome, ou nada equivalente. Não invente uma sintaxe sem confirmar na
documentação da ferramenta — é melhor entregar só os arquivos de fase organizados e avisar que a
automação de subagentes/orquestração, por enquanto, é a parte pensada especificamente para o
Claude Code.

## Na prática

Se o usuário disser qual ferramenta vai usar e não for Claude Code:

1. Gere sempre o kit "universal" da primeira seção acima.
2. Pesquise rapidamente a convenção de comando nomeado dessa ferramenta antes de gerar arquivo
   nela — não chute o caminho nem o formato de front matter.
3. Seja direto sobre o que não dá para replicar (orquestrador, subagentes) e ofereça a
   alternativa: os arquivos de fase organizados, prontos para colar um a um — o que já é uma
   melhora sobre o documento único de antes, mesmo sem a automação completa.
