# Referência — Kit de automação para Claude Code

> Use este arquivo quando o Passo 7 do SKILL.md pedir para gerar o kit de arquivos, além do
> documento do Passo 2. Todo o conteúdo abaixo é sobre a ferramenta (Claude Code) — o que muda
> por projeto é o conteúdo de cada fase e a lista de subagentes, não o formato.
>
> Baseado na documentação oficial de comandos/skills (`code.claude.com/docs/en/slash-commands`) e
> de subagentes (`code.claude.com/docs/en/sub-agents`). Se algo aqui parecer desatualizado ao
> gerar um kit — nomes de campo, comportamento de `$ARGUMENTS`, etc. — confirme lá antes de seguir
> por este arquivo; essas convenções mudam com as versões da ferramenta.

## Onde cada arquivo vai

```
<raiz do repo>/
├── CLAUDE.md                          # contexto fixo (Bloco A do plan_template.md)
├── docs/
│   ├── AUDIT.md                       # Fase 0 preenche
│   ├── DECISIONS.md                   # Fase 0 em diante
│   └── PROGRESS.md                    # checklist de fases — formato abaixo; é o que o orquestrador lê
└── .claude/
    ├── commands/
    │   ├── fase-0.md                  # um arquivo por fase/subfase do roadmap
    │   ├── fase-1a.md
    │   ├── fase-1b.md
    │   ├── ...
    │   └── proximo.md                 # orquestrador
    └── agents/
        ├── <papel-1>.md               # um por papel especializado recorrente no roadmap
        └── <papel-2>.md
```

Nomeie cada fase de forma **idêntica** em três lugares: no roadmap do documento do Passo 2, no
checklist do `docs/PROGRESS.md`, e no nome do arquivo em `.claude/commands/`. Use `fase-0`,
`fase-1a`, `fase-1b`, `fase-2`... (minúsculas, sem espaço) — é literalmente o nome do comando
(`/fase-1a`), e o orquestrador compara essas strings; uma inconsistência aqui quebra tudo
silenciosamente, sem erro visível.

`.claude/commands/*.md` é o formato mais simples para isto (um arquivo só, sem pasta própria) e
continua totalmente suportado — é equivalente a criar cada fase como uma skill em
`.claude/skills/<fase>/SKILL.md`, mas sem a pasta extra que só compensa quando a fase precisa de
arquivos de apoio além do prompt. Use `.claude/commands/` como padrão; use `.claude/skills/` só se
alguma fase específica precisar carregar um arquivo extra (uma referência grande, um script).

## `docs/PROGRESS.md` — formato do checklist

Tem que ser isto, sem variação, porque `/proximo` faz busca de texto nele:

```markdown
# Progresso

## Fases
- [ ] fase-0 — Auditoria inicial
- [ ] fase-1a — [nome do pilar de risco 1]
- [ ] fase-1b — [nome do pilar de risco 2]
- [ ] fase-1c — [fluxo core ponta a ponta]
- [ ] fase-2 — [primeira funcionalidade]

## Log
[Cada fase concluída ganha uma entrada aqui, adicionada pelo próprio agente ao final da fase:]
- fase-0 concluída em [data]. PR: [link]. Notas: [o que ficou pendente, se algo].
```

Marque `[x]` só depois que a fase estiver de fato concluída (PR aberto) — é essa marcação que diz
ao orquestrador que pode passar para a próxima.

## Arquivo de cada fase — `.claude/commands/<fase>.md`

Frontmatter fixo — `disable-model-invocation: true` porque cada fase tem efeito colateral real
(edita código, abre PR): só o humano decide quando rodar, nunca o Claude por conta própria dentro
de uma conversa qualquer.

```markdown
---
description: [uma linha — o que esta fase faz]
disable-model-invocation: true
---

Leia docs/PROGRESS.md e docs/DECISIONS.md antes de começar.
[Se depender de fase anterior:] Confirme que fase-[N anterior] está marcada como concluída em
docs/PROGRESS.md. Se não estiver, pare e avise em vez de prosseguir.

Objetivo desta etapa: [uma frase].

Tarefas:
1. [tarefa concreta e verificável]
2. [tarefa concreta e verificável]
...

[Se algum passo tiver um papel especializado correspondente em .claude/agents/, diga:]
Use o subagente `<papel>` para [a parte que ele cobre] — ele já sabe como fazer isso, não repita
o contexto todo no pedido, só passe o que for específico desta tarefa.

Regras:
- [o que explicitamente NÃO fazer nesta fase]
- Branch: `fase-[N]/[slug]`. Commits em Conventional Commits. PR ao final.
- Ao final, atualize docs/PROGRESS.md: marque esta fase como `[x]` e adicione uma linha no Log
  com a data, o link do PR, e qualquer coisa que ficou pendente.

Ao final, resuma para quem está acompanhando: o que foi feito, o link do PR, e a próxima fase
pendente segundo docs/PROGRESS.md.
```

Por que "leia docs/PROGRESS.md" em vez de colar o conteúdo dele aqui dentro: este arquivo pode
rodar numa sessão nova a qualquer momento, sem o resto desta conversa por perto — ele precisa
buscar o estado atual na hora, não confiar numa cópia que pode estar desatualizada.

## Orquestrador — `.claude/commands/proximo.md`

Este arquivo é quase todo fixo entre projetos — a única coisa que varia é o nome dos arquivos de
fase, e ele nem precisa saber esses nomes de antemão porque lê tudo de `docs/PROGRESS.md` na hora.
Gere-o assim:

```markdown
---
description: Executa a próxima fase pendente do plano, ou as fases indicadas em $ARGUMENTS, uma por vez, com checkpoint humano entre elas.
disable-model-invocation: true
---

Você está orquestrando as fases deste projeto. Antes de tudo, leia docs/PROGRESS.md.

Determine a lista de fases a executar nesta chamada:
- Se $ARGUMENTS tiver nomes de fase (ex.: "fase-0 fase-1a"), use exatamente essa lista, nessa
  ordem — não adicione nem remova nenhuma.
- Se $ARGUMENTS estiver vazio, use uma lista com um único item: a primeira fase ainda marcada
  como `[ ]` no checklist de docs/PROGRESS.md, na ordem em que aparecem lá.

Para cada fase da lista, nesta ordem:

1. Confirme que toda fase da qual ela declaradamente depende já está `[x]` em docs/PROGRESS.md.
   Se não estiver, pare aqui e avise — não prossiga por conta própria.
2. Leia o arquivo `.claude/commands/<fase>.md` inteiro e siga as instruções dele à risca,
   exatamente como se tivesse sido invocado diretamente com `/<fase>` — não resuma o conteúdo,
   não pule nenhuma tarefa listada nele.
3. Confirme que, ao final, docs/PROGRESS.md foi atualizado (checkbox marcado, entrada no Log) e
   que existe um PR aberto para esta fase. Se o arquivo da fase pedir Plan Mode, apresente o plano
   e espere aprovação antes de executar — não pule esse checkpoint.
4. Pare e reporte: nome da fase concluída, link do PR, o que ficou pendente (se algo). Se ainda
   houver fases na lista desta chamada, pergunte se deve seguir para a próxima ou se prefere
   revisar o PR primeiro. Só siga direto sem perguntar se o pedido original já dizia
   explicitamente para não parar entre fases dessa lista.

Nunca execute uma fase que não esteja no roadmap de docs/PROGRESS.md, mesmo que pareça uma boa
ideia — se faltar alguma coisa, avise e pare em vez de inventar uma fase nova.
```

Ponto importante para explicar ao usuário ao entregar: este comando não faz merge de PR nem pula a
revisão de Plan Mode — ele só evita que o humano precise copiar e colar cada prompt manualmente, e
evita esquecer uma fase no meio do caminho, porque a lista de fases vem sempre de
`docs/PROGRESS.md`, nunca da memória da conversa.

## Subagentes — `.claude/agents/<papel>.md`

Crie um por papel especializado que se repete no roadmap (ex.: backend, frontend, infraestrutura,
QA/testes, auditoria). Não crie um subagente para um papel que aparece uma única vez — o overhead
de manter o arquivo não compensa.

Regra de `tools`:
- Papéis que só leem e avaliam (auditoria, QA/revisão): `tools: Read, Grep, Glob, Bash` — sem
  `Edit`/`Write`.
- Papéis que implementam: `tools: Read, Edit, Write, Grep, Glob, Bash` (ajuste à lista real de
  ferramentas que o projeto usa, incluindo MCPs relevantes).
- Escolha `model` pelo custo/complexidade do papel: `haiku` para tarefas mecânicas e repetitivas,
  `sonnet` (ou `inherit`, para seguir o modelo da sessão principal) para implementação de lógica
  de produto.

```markdown
---
name: [papel]
description: [quando delegar a este subagente — específico, para não colidir com outro papel]
tools: [lista]
model: [sonnet | haiku | inherit]
---

Você é [papel] neste projeto. Leia CLAUDE.md e docs/DECISIONS.md antes de qualquer tarefa.

[2 a 5 frases sobre convenções específicas deste papel neste projeto — não repita o que já está
em CLAUDE.md, só o que é específico deste papel.]

Ao terminar, devolva um resumo curto (o que foi feito, arquivos tocados, qualquer decisão que
valha registrar em docs/DECISIONS.md) — quem te chamou vai repassar isso, não fale direto com o
usuário.
```

Um subagente, por padrão, já começa sem o histórico da conversa principal — só recebe `CLAUDE.md`,
a tarefa delegada e o próprio prompt acima. É esse comportamento nativo que atende ao pedido de
"subagente sem muito contexto"; não é preciso configurar nada além do arquivo em si para isso.

## Ao entregar o kit

- Gere todos os arquivos de uma vez, na estrutura acima.
- Não invoque nenhum comando nem rode `/proximo` você mesmo — a skill só prepara o kit, quem
  decide quando rodar cada fase é o usuário.
- Resuma em poucas linhas quais arquivos foram criados e como começar: "salve esta pasta na raiz
  do repositório, abra o Claude Code lá, e rode `/fase-0` para começar" (ou `/proximo` se quiser
  deixar o Claude Code decidir a próxima fase automaticamente).
