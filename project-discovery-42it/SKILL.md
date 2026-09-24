---
name: project-discovery-42it
description: 'Explora e documenta o estado atual de um projeto/código já existente, ANTES de qualquer plano de fases ser criado — sem propor solução, sem criar roadmap, sem tocar em código. Produz `docs/AUDIT.md`: stack, como cada área central funciona hoje (autenticação, dados, fluxo core), convenções observadas, pontos fracos ou incompletos encontrados, e decisões técnicas aparentes no código para confirmar depois. Use esta skill quando o usuário quiser entender, mapear ou documentar o que já existe num projeto antes de decidir o que fazer a seguir — "faz uma auditoria do projeto atual", "documenta o que já existe antes da gente planejar a próxima versão", "preciso de um raio-x do código antes de pedir uma mudança", "não sei bem qual é o estado atual desse repositório", "explora este projeto pra mim". NÃO use se o usuário já veio com um pedido concreto de mudança/feature/nova versão e quer um plano de fases para executá-lo — nesse caso use a skill agent-orchestration-plan-42it diretamente; o Passo 1 dela já lê `docs/AUDIT.md` se existir, ou explora o projeto do zero se não existir.'
---

# Project Discovery

## Por que isso existe

A skill `agent-orchestration-plan-42it` já sabe ler um projeto existente como parte do seu Passo 1,
mas isso significa reexplorar o código toda vez que se quer planejar uma mudança nova — e a
exploração fica amarrada a um plano específico, mesmo quando o que a pessoa queria, naquele
momento, era só entender o que já existe. Esta skill separa essa etapa: faz a exploração uma
única vez, sem compromisso com nenhuma fase ou plano ainda, e produz um documento
(`docs/AUDIT.md`) que qualquer plano futuro — desta ou de outra ferramenta — pode ler direto, sem
repetir o trabalho de descoberta do zero.

O resultado desta skill é sempre **um documento de estado atual, nunca uma proposta de solução,
roadmap ou fase**. Decidir o que fazer com o que foi encontrado é trabalho de outra conversa,
normalmente com `agent-orchestration-plan-42it`.

## Quando usar (e quando não)

- **Use** quando o projeto já existe (não é greenfield) e ainda não tem `docs/AUDIT.md`, ou o que
  existe está claramente desatualizado, e o usuário ainda não tem um pedido de mudança específico
  em mãos — só quer entender/documentar o estado atual antes de decidir o próximo passo.
- **Não use** se o usuário já veio com uma mudança, feature ou nova versão específica para
  planejar. Nesse caso vá direto para `agent-orchestration-plan-42it` — o Passo 1 dela já cobre a
  leitura do projeto (existente ou não) como parte da própria conversa de planejamento, e se
  `docs/AUDIT.md` já existir (porque esta skill rodou antes), ela usa esse documento como ponto de
  partida em vez de reexplorar. Rodar as duas skills em sequência para o mesmo pedido é trabalho
  duplicado.
- Funciona tanto aqui, numa conversa com acesso a arquivo/bash, quanto instalada dentro do próprio
  Claude Code (`.claude/skills/project-discovery-42it/` ou `~/.claude/skills/project-discovery-42it/`) rodando
  num projeto real em disco — as instruções abaixo usam só operações genéricas de leitura (listar
  arquivos, ler conteúdo, grep, `git log`), disponíveis nos dois ambientes.

## Passo 1 — Explorar o projeto (só leitura)

Nunca edite código nem rode qualquer comando que altere o repositório nesta skill — é 100%
leitura. Cubra:

- **Estrutura e stack**: árvore de diretórios em alto nível; linguagens e frameworks (via
  `package.json`, `pyproject.toml`, `go.mod`, etc.); dependências e versões relevantes.
- **Documentação já existente**: README, `docs/`, ADRs, comentários de arquitetura — leia antes de
  inferir algo que já está escrito em algum lugar.
- **Áreas centrais**: como autenticação/autorização, dados/armazenamento, e o fluxo core do
  produto funcionam hoje, com base no código real — não no que o README promete.
- **Convenções em uso**: naming, organização de pastas, padrão de testes, linter/formatter
  configurado, convenção de commit já praticada (mesmo que informal).
- **Sinais de atividade e dívida**: `git log` recente para entender o que mudou por último; busca
  por `TODO`/`FIXME`/`HACK`; qualquer coisa que pareça inconsistente, incompleta, ou meio-feita.

Registre o que encontrar por área, com referência a arquivo/trecho quando possível — evite
generalização vaga tipo "o código está desorganizado" sem dizer onde e por quê.

## Passo 2 — Perguntas objetivas (no máximo 2 a 3)

Só pergunte o que não dá para descobrir lendo o código: decisões de negócio, coisas que parecem
inconsistentes no código mas podem ser intencionais, prioridade entre os pontos fracos
encontrados (se for relevante para o documento). Não pergunte o que já foi possível inferir com
razoável confiança — registre como achado, não como pergunta.

## Passo 3 — Escrever `docs/AUDIT.md`

Use `references/audit_template.md` como esqueleto. Se já existir um `docs/AUDIT.md` no projeto,
trate como atualização: leia o que já tem, preserve o que ainda é válido, e ajuste/complete em vez
de sobrescrever sem necessidade — a menos que o usuário peça explicitamente uma reauditoria
completa do zero.

Seções obrigatórias (detalhe exato em `references/audit_template.md`):
1. Visão geral do projeto.
2. Como cada área central funciona hoje.
3. Convenções observadas.
4. Pontos fracos, incompletos ou inconsistentes — sem prescrever solução; isso é trabalho de um
   plano depois, não desta auditoria.
5. Decisões técnicas aparentes no código, marcadas para confirmação — não escreva em
   `docs/DECISIONS.md` a partir daqui; isso fica para quando um plano de fato existir (ver
   Passo 5), porque uma decisão só devia ser registrada como decisão depois que alguém a confirma.
6. Perguntas em aberto que não foram respondidas.

## Passo 4 — Qualidade

- Auditoria sem opinião prematura: registre o que existe e o que parece frágil, mas não decida "a
  solução é X" — isso é trabalho do planejamento depois, com o usuário no meio da decisão.
- Seja factual e específico (arquivo, trecho, comando rodado) em vez de generalização.
- Se a base de código for grande, não tente cobrir tudo com o mesmo nível de detalhe — priorize as
  áreas que um plano futuro provavelmente vai tocar (autenticação, dados, fluxo core) sobre
  detalhes de módulos periféricos, e diga explicitamente o que ficou de fora por escopo.

## Passo 5 — Entregar

- Entregue só `docs/AUDIT.md`. Não crie `CLAUDE.md`/`AGENTS.md`, não crie `docs/DECISIONS.md` nem
  `docs/PROGRESS.md`, e não gere nenhum arquivo de comando ou subagente — tudo isso é trabalho da
  skill `agent-orchestration-plan-42it`, que deve ser usada depois, quando o usuário já tiver uma
  mudança específica para planejar.
- Não proponha roadmap nem fases — isso ainda não é um plano, é só o retrato do que existe hoje.
- Ao final, diga isso ao usuário em uma frase: "quando você tiver uma mudança específica para
  planejar, é só pedir — a skill de orquestração de plano já vai partir desta auditoria em vez de
  reexplorar o projeto do zero."
