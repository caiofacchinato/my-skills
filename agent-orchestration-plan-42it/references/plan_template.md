# Template — Plano de Orquestração de Agente de IA

> Este é o esqueleto a reutilizar em todo plano gerado por esta skill. Trechos entre `[colchetes]`
> são para preencher com o conteúdo real do projeto. Trechos sem colchetes (a explicação de Plan
> Mode, a convenção de git, a nota final sobre subagentes) são texto de ferramenta/processo — quase
> não mudam de projeto para projeto, reaproveite como está, só ajustando o que for claramente
> específico da ferramenta que o usuário vai usar (ex.: se não for Claude Code, adapte o nome do
> atalho de Plan Mode).
>
> Este documento é o mesmo nos dois modos de entrega (ver Passo 6 do SKILL.md). A única diferença é
> a seção 4: em modo documento, cada prompt de fase é escrito aqui, no corpo do documento, para
> copiar e colar; em modo kit, esta seção vira só uma lista curta (nome da fase + uma frase) porque
> o prompt completo já mora no arquivo da fase (`docs/fases/<fase>.md`) — ver
> `references/claude_code_kit.md`. Em modo kit, o Bloco A também recebe a seção "Protocolo de
> fases" (texto pronto no mesmo arquivo), que é o que permite ao usuário só dizer "próximo".

---

## Cabeçalho do documento

```markdown
# Plano de Orquestração de Agente de IA — [NOME DO PROJETO]

> Este documento é para o humano usar, não para o agente. Ele contém: (1) o contexto fixo do
> produto, que deve virar um arquivo de memória no repositório, e (2) o roadmap de fases, em ordem,
> com os prompts prontos para colar (modo documento) ou já como arquivos de fase prontos para o
> repositório (modo kit) na [FERRAMENTA: Claude Code / Gemini Antigravity / OpenCode / outra].
```

## 0. Como usar isto na prática

```markdown
1. Confirme que o projeto está num repositório git com remoto no GitHub (a Fase 0 já assume isso;
   se for um projeto greenfield sem repositório ainda, a Fase 0 cria o repositório).
2. Copie o Bloco A abaixo e salve como `CLAUDE.md` na raiz do repo (ou o arquivo de memória
   equivalente da ferramenta escolhida — ex.: `AGENTS.md` é aceito por várias delas). Se você
   recebeu o kit de arquivos junto com este documento, isso já vem pronto — só copiar a pasta.
3. Rode os prompts em ordem, um por sessão (contexto limpo a cada um). Não pule etapas. Se você
   recebeu o kit de arquivos, basta abrir a ferramenta agentic no repositório e dizer algo natural:
   "começar fase-0", "1a", ou só "próximo" para deixá-la decidir a próxima fase pendente sozinha.
   Ela confirma em uma linha qual fase vai executar antes de agir.
4. Depois de cada PR gerado pelo agente, revise você mesmo. Ao final de cada fase o agente pergunta
   se ela está ok: se você responder que sim, ele faz o merge do PR e limpa as branches da fase
   (local e remota) sozinho, deixando `main` atualizada para a próxima; se pedir ajustes, ele
   corrige na mesma branch e pergunta de novo. Não encadeie tudo automaticamente — cada fase é um
   ponto de checagem humana, e sem o seu "ok" não há merge. O protocolo do kit também
   respeita isso: o agente para e pergunta entre uma fase e outra, mesmo quando você pede várias de
   uma vez.
5. Cada prompt já instrui o agente a atualizar `docs/PROGRESS.md`. É esse arquivo que evita perder
   o fio da meada entre sessões — e, em modo kit, é o mesmo arquivo que o Protocolo de fases lê para
   saber qual é a próxima fase.
```

## Bloco A — Arquivo de contexto fixo (salvar como `CLAUDE.md`)

Preencha com o conteúdo real do produto. Estrutura mínima:

```markdown
# Contexto do Projeto — [NOME DO PRODUTO]

## O que é
[Descrição do produto em poucas frases — o problema que resolve e para quem.]

## Premissas fixas (não questionar nem "corrigir")
[Lista curta de decisões conscientes de negócio/arquitetura que o agente não deve reinterpretar.
Ex.: modelo de tenancy escolhido, provedor já decidido, restrição de prazo. Cada item deve ser
específico o bastante para ser verificável — não "seja simples", mas "uma instância por cliente,
sem multi-tenant compartilhado por enquanto".]

## Princípios de simplicidade / preferências técnicas
[Se houver. Ex.: preferir serviços gerenciados a infra própria; não otimizar para uma escala que
ainda não existe.]

## Estado atual conhecido (antes da auditoria da Fase 0)
[O que se sabe hoje: escopo reduzido recentemente? o quê está fraco/incompleto? Se não se sabe
ainda o estado técnico real, diga isso explicitamente — é o objetivo da Fase 0 descobrir.]

## Como trabalhar neste repositório
- Leia sempre docs/PROGRESS.md e docs/DECISIONS.md antes de começar qualquer tarefa nova.
- Uma branch por fase/subfase, commits em Conventional Commits, um PR para main ao final de cada
  etapa. Ao final de cada etapa, pergunte ao usuário se ela está ok; só com resposta positiva
  explícita, faça o merge do PR e apague a branch da fase (local e remota).
- Nunca commitar segredos (.env, chaves de API, credenciais reais).
- Ao final de cada sessão, atualizar docs/PROGRESS.md com o que foi feito, o que falta, e
  problemas encontrados.
- Não expandir escopo por conta própria.
```

## 0.1 Plan Mode vs. modo de execução

```markdown
A maioria das ferramentas agentic tem dois modos de operação — os nomes variam, mas o conceito é
o mesmo:

- Modo de planejamento (no Claude Code: `Shift+Tab` duas vezes, ou `/plan`, ou
  `claude --permission-mode plan`): o agente só lê o código e escreve um plano — fica bloqueado de
  editar arquivos, rodar comandos ou commitar até você aprovar.
- Modo de execução (padrão): o agente já pode escrever, editar, rodar comandos e commitar direto.

Regra prática:
- Prompt de auditoria/diagnóstico (Fase 0): pode rodar direto em modo de execução — já é
  restrito a criar só documentação, nunca código de aplicação.
- Prompts de implementação: comece em modo de planejamento. Revise o plano proposto (quais
  arquivos serão tocados, abordagem, riscos), aprove ou ajuste, e só então deixe o agente executar,
  commitar e abrir o PR. Isso importa mais em migrações de auth/storage/schema.
- Se a ferramenta não for o Claude Code, procure o equivalente — o princípio ("plano primeiro,
  execução depois com aprovação humana no meio") é o que importa, não o atalho específico.
```

## 1. Arquivos de memória

```markdown
| Arquivo | Criado em | Para que serve |
|---|---|---|
| `CLAUDE.md` (ou equivalente) | Você, no início | Contexto fixo do produto, quase nunca muda |
| `docs/AUDIT.md` | Fase 0 | Raio-x do estado atual comparado às premissas |
| `docs/DECISIONS.md` | Fase 0 em diante | Registro de decisões técnicas (tipo ADR) e por quê |
| `docs/PROGRESS.md` | Fase 0 em diante | Checklist vivo: o que está pronto, o que falta |
```

## 2. Roadmap resumido

Preencha o roadmap real do projeto seguindo a regra do Passo 3 do SKILL.md. Formato:

```markdown
- Fase 0 — [Auditoria / Setup inicial]
- Fase 1 — Fundação
  - 1A. [pilar de risco 1]
  - 1B. [pilar de risco 2]
  - 1C. [fluxo core ponta a ponta, unindo as peças anteriores]
- Fase 2 — [primeira funcionalidade construída sobre a fundação]
- Fase 3+ (backlog, não detalhar agora) — [itens fora de escopo por ora]

Critério de pronto da Fase 1: [uma frase objetiva e testável].
```

Os rótulos acima ("Fase 0", "1A"...) são para leitura humana no documento. **Em modo kit**, cada
um também precisa de um slug idêntico em minúsculas e com hífen — `fase-0`, `fase-1a`, `fase-1b`,
`fase-2` — que é o nome real do arquivo em `docs/fases/` e a string usada no checklist de
`docs/PROGRESS.md`. Defina o slug de cada fase aqui mesmo, junto do roadmap, e reuse-o sem
variação em todo o resto do kit (ver `references/claude_code_kit.md`).

## 3. Convenção de Git/GitHub

```markdown
- `main` protegida. Uma branch por fase/subfase, nomeada de forma previsível
  (`phase-0/...`, `phase-1/...`, `phase-2/...`).
- Commits em Conventional Commits (feat:, fix:, docs:, refactor:, chore:, test:).
- Um PR por etapa, com descrição do que mudou, o que foi testado manualmente, e riscos conhecidos.
- Encerramento de etapa: o agente pergunta se a etapa está ok. Com resposta positiva explícita, ele
  faz o merge do PR (squash, salvo decisão em contrário em `docs/DECISIONS.md`), confirma que o
  estado é `MERGED`, volta para `main` atualizada e apaga a branch da etapa, local e remota. Sem
  resposta positiva (ajustes pedidos, resposta ambígua), não há merge nem limpeza. Se o merge for
  bloqueado (checks, review exigido, conflito), o agente para e reporta — nunca contorna a proteção
  de `main`.
- Nenhum segredo commitado — usar `.env.example` sem valores reais.
```

## 4. Formato de cada prompt de fase

**Em modo documento**, todo prompt de fase, sem exceção, segue este esqueleto, escrito aqui no
corpo do documento:

```markdown
### PROMPT [N] — [Nome da fase]

**Modo recomendado: [execução direto / comece em Plan Mode].** [uma frase de porquê]

```
Leia [arquivos de memória relevantes] antes de começar. [Se depende de fase anterior: confirme que
está concluída lendo docs/PROGRESS.md; se não estiver, pare e avise em vez de prosseguir.]

Objetivo desta etapa: [uma frase].

Tarefas:
1. [tarefa concreta e verificável]
2. [tarefa concreta e verificável]
...

Regras:
- [o que explicitamente NÃO fazer nesta fase — delimitar escopo]
- Branch: `[nome]`. Commits Conventional Commits. PR ao final.
- Atualize docs/PROGRESS.md marcando o que foi concluído e o que ficou pendente (dentro do PR).

Ao final da implementação, resuma: [o que você quer que o agente reporte de volta] e pergunte
explicitamente se a etapa está ok para ser encerrada.

Encerramento — só depois de resposta positiva explícita: faça o merge do PR (`gh pr merge <PR>
--squash --delete-branch`), confirme que o estado é `MERGED`, volte para `main` (`git checkout main
&& git pull --ff-only && git fetch --prune`) e apague a branch local desta etapa se ainda existir.
Se o merge for bloqueado, pare e reporte. Se o usuário pedir ajustes, não faça merge: ajuste na
mesma branch e pergunte de novo.
```
```

Use este esqueleto para cada prompt do roadmap (Fase 0, cada subfase da Fase 1, e cada fase de
funcionalidade seguinte).

**Em modo kit**, este mesmo esqueleto vira o conteúdo de `docs/fases/<fase>.md` (formato exato em
`references/claude_code_kit.md`) — não escreva o prompt aqui também. Aqui, nesta seção do
documento, entra só uma lista curta apontando para cada arquivo:

```markdown
- `fase-0` — [uma frase]. Arquivo: `docs/fases/fase-0.md`.
- `fase-1a` — [uma frase]. Arquivo: `docs/fases/fase-1a.md`.
...
```

## 5. Pontos em aberto

```markdown
Liste aqui os riscos de produto identificados mas não mencionados pelo usuário (privacidade,
moderação de conteúdo, custo de infraestrutura por cliente/uso, segurança de identificadores
públicos, dados pessoais/LGPD, etc.). Mesmo que a lista fique curta, não omita esta seção.
```

## 6. Nota final sobre orquestração avançada

```markdown
Ferramentas como o Claude Code permitem subagentes especializados (um para backend, um para
frontend, um para QA). Usar um subagente para isolar contexto — ele roda uma tarefa específica
com um mínimo de contexto e devolve um resumo para esta sessão — é seguro desde a primeira fase
e está incluído no kit de arquivos deste plano, quando gerado (ver seção de subagentes). O que fica para depois é rodar várias fases *em paralelo*, editando código ao mesmo
tempo: para a reestruturação inicial, a recomendação é manter as fases sequenciais e revisadas
pelo humano a cada PR — é mais fácil de auditar enquanto a base de código ainda está instável.
```
