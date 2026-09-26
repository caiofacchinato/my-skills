# Referência — Kit de automação para Claude Code

> Use este arquivo quando o Passo 7 do SKILL.md pedir para gerar o kit de arquivos, além do
> documento do Passo 2. O kit funciona por **contexto**, não por comando nomeado: o usuário abre uma
> sessão nova e diz algo natural ("próximo", "começar fase-1c") e o agente resolve a fase a partir
> de um protocolo escrito no `CLAUDE.md`. O que muda por projeto é o conteúdo de cada fase e a lista
> de subagentes, não o formato.
>
> Baseado na documentação oficial de memória/`CLAUDE.md` e de subagentes
> (`code.claude.com/docs/en/sub-agents`). Se algo aqui parecer desatualizado ao gerar um kit —
> nomes de campo dos subagentes, comportamento de `CLAUDE.md` etc. — confirme lá antes de seguir por
> este arquivo; essas convenções mudam com as versões da ferramenta.

## Onde cada arquivo vai

```
<raiz do repo>/
├── CLAUDE.md                          # contexto fixo (Bloco A) + Protocolo de fases (abaixo)
├── docs/
│   ├── AUDIT.md                       # Fase 0 preenche
│   ├── DECISIONS.md                   # Fase 0 em diante
│   ├── PROGRESS.md                    # checklist de fases — formato abaixo; é o que o protocolo lê
│   └── fases/
│       ├── fase-0.md                  # um arquivo por fase/subfase do roadmap
│       ├── fase-1a.md
│       ├── fase-1b.md
│       └── ...
└── .claude/
    └── agents/
        ├── <papel-1>.md               # um por papel especializado recorrente no roadmap
        └── <papel-2>.md
```

Cada fase tem um **slug** — `fase-0`, `fase-1a`, `fase-1b`, `fase-2`... (minúsculas, sem espaço) — e
ele precisa ser idêntico no roadmap do documento do Passo 2, no checklist do `docs/PROGRESS.md` e
no nome do arquivo em `docs/fases/`. O slug é a chave estável do kit. O usuário não precisa digitá-lo
certo (o agente resolve "1c", "fase 1c", "a da autenticação" por semelhança, ver Protocolo abaixo),
mas as três fontes precisam concordar entre si, senão o agente pode pular ou repetir fase sem
avisar.

Os arquivos de fase são markdown simples, sem front matter e sem ser comando: só são lidos quando a
fase é resolvida, então não pesam no contexto de início de sessão. Só o `CLAUDE.md` é carregado
sozinho a cada sessão nova — por isso o protocolo mora nele, e os prompts das fases não.

## `docs/PROGRESS.md` — formato do checklist

Tem que ser isto, sem variação, porque o protocolo faz busca de texto nele:

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

Marque `[x]` só depois que a fase estiver de fato concluída (PR aberto, com a implementação
pronta) — o agente faz essa marcação na própria branch da fase, antes de pedir a aprovação do
usuário, então ela chega em `main` junto com o merge do PR. É essa marcação que diz ao protocolo
que pode passar para a próxima. Se o usuário reprovar a fase, o agente desmarca o `[x]` enquanto
ajusta.

## Protocolo de fases — seção do `CLAUDE.md`

Este é o orquestrador do kit, escrito como instrução e não como comando. É praticamente fixo entre
projetos: cole no `CLAUDE.md` (logo depois do Bloco A) quase sem alteração. Ele precisa ficar
curto — `CLAUDE.md` é lido em toda sessão.

```markdown
## Protocolo de fases

Quando o usuário pedir para começar, continuar ou seguir uma fase — "próximo", "continuar",
"começar fase-1c", "1c", "seguir com a 0 e a 1", ou qualquer frase de sentido equivalente:

1. Leia docs/PROGRESS.md e determine a lista de fases a executar:
   - se o usuário citou fases (pelo slug, pelo número ou pelo nome), use exatamente essas, na ordem
     em que aparecem no checklist — não adicione nem remova nenhuma;
   - se não citou nenhuma ("próximo", "continuar"), use uma lista com um único item: a primeira
     fase ainda marcada `[ ]` no checklist.
   Se a frase não corresponder a nenhuma fase do checklist, ou corresponder a mais de uma, pergunte
   qual é — nunca invente uma fase que não esteja em docs/PROGRESS.md.
2. Antes de agir, confirme em uma linha qual fase vai executar e em que modo ("Vou executar
   fase-1c, começando em Plan Mode. Ok?") e espere o "ok". Toda fase tem efeito colateral real
   (edita código, abre PR, faz merge); esta confirmação impede que uma frase ambígua dispare uma.
3. Para cada fase da lista, na ordem:
   a. Confirme que toda fase da qual ela declaradamente depende está `[x]` em docs/PROGRESS.md.
      Se não estiver, pare e avise — não prossiga por conta própria.
   b. Leia docs/fases/<slug>.md inteiro e siga as instruções dele à risca — não resuma o conteúdo,
      não pule nenhuma tarefa. Se o arquivo pedir Plan Mode, apresente o plano e espere aprovação
      antes de executar.
   c. Ao final da implementação, confirme que docs/PROGRESS.md foi atualizado (checkbox marcado,
      entrada no Log) e que existe um PR aberto para a fase.
   d. Pare, reporte (fase, link do PR, o que ficou pendente) e pergunte se a fase está ok para ser
      encerrada. Esta pergunta nunca é pulada, nem quando o usuário pediu para não parar entre
      fases: sem uma resposta positiva explícita, não há merge.
   e. Com a resposta positiva, execute o encerramento descrito no próprio arquivo da fase (merge do
      PR, confirmação de `MERGED`, limpeza das branches da fase, `main` atualizada). Se o merge for
      bloqueado, pare e reporte — não avance para a próxima fase da lista.
   f. Só então, se ainda houver fases na lista, siga para a próxima a partir da `main` atualizada.
      Se o usuário pediu explicitamente para não parar entre fases, siga direto (repetindo a
      pergunta do passo d a cada fase); caso contrário, pergunte antes se deve seguir.
```

Ponto importante para explicar ao usuário ao entregar: este protocolo só faz o merge do PR e a
limpeza das branches depois que o usuário responde positivamente à pergunta "a fase está ok?", e
nunca pula a revisão de Plan Mode. Ele evita que o humano precise copiar e colar cada prompt
manualmente, mergear PR e apagar branch na mão, e evita esquecer uma fase no meio do caminho,
porque a lista de fases vem sempre de `docs/PROGRESS.md`, nunca da memória da conversa.

Trade-off a deixar claro: uma instrução no `CLAUDE.md` é menos determinística que um comando —
o agente pode interpretar mal a frase ou pular um passo. As mitigações estão no próprio protocolo:
a confirmação de uma linha antes de executar, Plan Mode nas fases de risco, e o "ok" explícito
antes do merge.

## Arquivo de cada fase — `docs/fases/<fase>.md`

Markdown simples, sem front matter. É a fonte única do prompt da fase.

```markdown
# [slug] — [nome da fase]

Leia docs/PROGRESS.md e docs/DECISIONS.md antes de começar.
[Se depender de fase anterior:] Confirme que fase-[N anterior] está marcada como concluída em
docs/PROGRESS.md. Se não estiver, pare e avise em vez de prosseguir.

Modo recomendado: [execução direto / comece em Plan Mode]. [uma frase de porquê]

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
- Ao final da implementação, atualize docs/PROGRESS.md: marque esta fase como `[x]` e adicione
  uma linha no Log com a data, o link do PR, e qualquer coisa que ficou pendente. Isso vai dentro
  do próprio PR.

Ao final da implementação, resuma para quem está acompanhando: o que foi feito, o link do PR, e o
que ficou pendente. Em seguida, pergunte explicitamente se a fase está ok para ser encerrada.

Encerramento — só depois de uma resposta positiva explícita ("ok", "aprovado", "pode mergear"):
1. Faça o merge do PR: `gh pr merge <PR> --squash --delete-branch`, ou a estratégia que
   CLAUDE.md/docs/DECISIONS.md definirem, se houver. Se o merge for bloqueado (checks falhando,
   review exigido, conflito), pare e reporte o motivo — nunca contorne com `--admin` nem force.
2. Confirme com `gh pr view <PR> --json state` que o estado é `MERGED`. Só então continue.
3. Limpe as branches desta fase, e só elas: `git checkout main`, `git pull --ff-only`,
   `git fetch --prune`, e apague a branch local da fase se ainda existir (`git branch -D
   fase-[N]/[slug]` — precisa de `-D` porque o squash não deixa a branch como "mergeada" para o
   git local). Confirme que a branch remota também foi removida.
4. Reporte: PR mergeado, branches removidas, `main` atualizada e a próxima fase pendente segundo
   docs/PROGRESS.md.

Se o usuário pedir ajustes, ou se a resposta for ambígua ou negativa, não faça merge nem limpeza:
ajuste na mesma branch, atualize o PR, e pergunte de novo.
```

Por que "leia docs/PROGRESS.md" em vez de colar o conteúdo dele aqui dentro: este arquivo pode ser
lido numa sessão nova a qualquer momento, sem o resto desta conversa por perto — ele precisa buscar
o estado atual na hora, não confiar numa cópia que pode estar desatualizada.

## Comandos nomeados (opcional)

Só gere se o usuário pedir um atalho tipo `/fase-1c` ou `/proximo`. Não são necessários: o
protocolo acima já cobre o mesmo uso, sem exigir que a pessoa lembre o nome exato.

Se pedirem, gere `.claude/commands/<slug>.md` como um arquivo fino que só aponta para a fonte
única, sem duplicar o prompt:

```markdown
---
description: [uma linha — o que esta fase faz]
disable-model-invocation: true
---

Execute a fase [slug] seguindo o Protocolo de fases do CLAUDE.md e as instruções de
docs/fases/[slug].md.
```

E, para o atalho `/proximo`, um `.claude/commands/proximo.md` equivalente com "Execute a próxima
fase pendente (ou as indicadas em $ARGUMENTS) seguindo o Protocolo de fases do CLAUDE.md." O
`disable-model-invocation: true` é o que impede o Claude de disparar uma fase por conta própria
dentro de uma conversa qualquer.

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
Contexto sozinho não substitui isso: é por isso que os subagentes continuam sendo arquivos mesmo
com o resto do kit funcionando só por contexto.

## Ao entregar o kit

- Gere todos os arquivos de uma vez, na estrutura acima.
- Não execute nenhuma fase você mesmo — a skill só prepara o kit, quem decide quando rodar cada
  fase é o usuário.
- Resuma em poucas linhas quais arquivos foram criados e como começar: "salve esta estrutura na raiz
  do repositório, abra o Claude Code lá, e diga 'começar fase-0' — ou só 'próximo' quando quiser
  que ele decida a próxima fase sozinho."
