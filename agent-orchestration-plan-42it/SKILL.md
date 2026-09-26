---
name: agent-orchestration-plan-42it
description: 'Cria ou atualiza um plano faseado de prompts para orquestrar agentes de codificação de IA (Claude Code, Gemini Antigravity, OpenCode etc.) num projeto novo ou existente, e pode gerar o kit de arquivos pronto para o repositório em vez de só um documento para copiar e colar: CLAUDE.md/AGENTS.md, memória entre sessões (AUDIT.md, DECISIONS.md, PROGRESS.md), um arquivo por fase/subfase, um protocolo no CLAUDE.md que permite dizer só "próximo" ou "começar fase-1c" e seguir o roadmap sem pular etapas, e subagentes com contexto isolado para tarefas especializadas. Use sempre que o usuário quiser planejar/organizar um projeto de software para construir ou refatorar com ajuda de IA, pedir "um prompt para o Claude Code", quebrar um projeto em fases, disser que o código está desorganizado e precisar de um plano pra arrumar com IA, anexar documentos/código pedindo pra organizar ou preparar prompts, pedir pra criar/atualizar CLAUDE.md, AGENTS.md, PROGRESS.md ou DECISIONS.md, ou pedir para automatizar/deixar pronto o kit de fases e subagentes para rodar direto na ferramenta agentic. Dispare mesmo sem a palavra "orquestração" — "me ajuda a planejar isso pro Claude Code" já basta.'
---

# Agent Orchestration Plan

## Por que isso existe

Projetos de software tocados por agentes de codificação de IA (Claude Code e afins) costumam
quebrar de três formas: (1) o agente perde ou reinterpreta as premissas do produto de sessão para
sessão, porque nada fica fixado por escrito; (2) uma mudança arriscada (schema, autenticação,
storage) é executada de uma vez sem checkpoint humano; (3) sem convenção de branch/commit/PR, o
histórico vira um emaranhado difícil de auditar. Esta skill existe para produzir, antes de
qualquer código ser tocado, um documento que resolve os três problemas de uma vez: um contexto
fixo, memória entre sessões, fases pequenas com prompts prontos, e uma regra clara de quando pausar
para revisão humana.

O resultado desta skill é sempre **um plano — e, quando fizer sentido, o kit de arquivos que
coloca esse plano pra funcionar na ferramenta agentic —, nunca a execução das fases em si**.
A skill nunca roda um prompt de fase, nunca abre PR, nunca decide "seguir para a próxima
fase" por conta própria: isso é sempre decisão do usuário, mesmo quando o kit inclui um
protocolo de fases que automatiza a mecânica de seguir o roadmap sem pular etapas.

Se o usuário ainda não tem uma mudança específica em mãos — só quer entender o estado atual de um
projeto existente antes de decidir o que fazer — isso é trabalho da skill irmã
`project-discovery-42it`, não desta aqui. Esta skill assume que já existe um pedido de mudança
(mesmo que vago) para transformar em plano; se `docs/AUDIT.md` já existir porque a outra skill
rodou antes, use-o (ver Passo 1) em vez de reexplorar o projeto do zero.

## Passo 1 — Entender o pedido

Antes de montar o plano, você precisa de:

1. **O que é o produto/projeto**, em poucas frases.
2. **As premissas fixas** — coisas que o agente de codificação NÃO deve reinterpretar ou "corrigir"
   por conta própria (modelo de negócio, decisões de arquitetura já tomadas conscientemente,
   restrições de prazo/orçamento). É comum o usuário só falar isso de forma corrida — sua tarefa é
   destilar em frases curtas e inegociáveis.
3. **Estado atual**: já existe código? Passou por alguma mudança de escopo recente? O que está
   fraco, incompleto ou quebrado hoje?
4. **Princípios de simplicidade/preferências técnicas**, se houver (ex.: "prefiro serviço
   gerenciado a infraestrutura própria", "sem pressa para escalar agora").
5. Se é um plano **novo** ou uma **atualização/aprimoramento** de um plano já existente.
6. **Qual ferramenta agentic vai rodar o plano** (Claude Code, Gemini Antigravity, OpenCode,
   outra) e **se o usuário quer só o documento para copiar/colar ou o kit de arquivos** pronto
   para o repositório (ver Passo 6). Não precisa perguntar isso de forma isolada — normalmente
   já vem junto do pedido ("prepara pro Claude Code já rodar", "só quero o plano pra eu ler").
   Se não vier e o projeto tiver mais de uma fase, o padrão é: Claude Code + kit de arquivos
   (é o caso mais comum e o que resolve o problema de copiar prompt por prompt); se a sessão
   não tiver como escrever arquivo nenhum, caia para o documento único mesmo sem perguntar.

Como obter isso:

- **Antes de mais nada, confira se já existe `docs/AUDIT.md`** no projeto (gerado pela skill
  `project-discovery-42it`, rodada antes, ou por uma Fase 0 anterior). Se existir, leia-o e use-o como
  a fonte do item 3 (estado atual) em vez de reexplorar o código do zero — ele já traz stack,
  convenções, e pontos fracos encontrados. Trate qualquer "decisão técnica aparente a confirmar"
  listada ali como pergunta candidata para o usuário, em vez de assumir.
- Se o usuário anexou documentos, código, ou um plano anterior (inclusive um gerado por esta
  mesma skill numa conversa passada), **leia tudo primeiro** e extraia o máximo possível daí antes
  de perguntar qualquer coisa — não peça para o usuário reescrever o que já está nos anexos.
- Se for uma **atualização** de um plano existente, trate o documento anexado como a fonte de
  verdade da estrutura: preserve as seções e convenções que já estão lá, e ajuste só o que foi
  pedido (nova fase, prompt revisado, premissa alterada) em vez de regenerar do zero.
- Se depois de ler tudo ainda faltar algo essencial do item 1 ou 2 (o que é o produto, ou quais são
  as premissas inegociáveis), pergunte direto ao usuário, em linguagem natural — no máximo 2 ou 3
  perguntas objetivas. Não vire um formulário longo.
- Para o resto (detalhes menores, riscos de produto não mencionados), não bloqueie por isso: registre
  como suposição ou como item na seção "Pontos em aberto" do documento final (ver Passo 2).

## Passo 2 — Estrutura do documento

Use `references/plan_template.md` como esqueleto. Ele já traz, prontas e testadas, as seções que
não mudam de projeto para projeto (a explicação de Plan Mode, a convenção de git, o formato de
prompt por fase). O que muda por projeto é: as premissas do Bloco A, o roadmap de fases, o
conteúdo de cada prompt, e os pontos em aberto.

A ordem das seções no documento final é sempre:

1. Bloco de contexto fixo (vira `CLAUDE.md`/`AGENTS.md`) com as premissas inegociáveis do produto.
2. Explicação de Plan Mode vs. modo de execução (reaproveite o texto do template quase verbatim —
   é conceito de ferramenta, não de projeto).
3. Tabela dos arquivos de memória entre sessões (`AUDIT.md`, `DECISIONS.md`, `PROGRESS.md`).
4. Roadmap resumido por fases (ver Passo 3).
5. Convenção de Git/GitHub (branches, commits, PR — reaproveite do template).
6. Um prompt pronto por fase/subfase, cada um anotado com o modo recomendado (ver Passo 4).
7. "Pontos em aberto" — nunca pule esta seção, mesmo que fique com 1 ou 2 itens (ver Passo 5).
8. Nota curta sobre orquestração avançada. Distinga as duas coisas: subagentes para isolar
   contexto (um papel especializado que roda com o mínimo de contexto necessário e devolve um
   resumo) são seguros e recomendados desde já — em modo kit, o Passo 7 já os inclui; já rodar
   várias fases *em paralelo* editando código ao mesmo tempo é o que fica para depois que o fluxo
   sequencial estiver rodando e estável, porque é mais difícil auditar enquanto a base ainda muda.

## Passo 3 — Como quebrar em fases

Regra geral, válida para qualquer domínio de produto:

- **Fase 0 é sempre uma auditoria/diagnóstico**, só leitura, sem alterar código de aplicação —
  varrer o que existe, comparar com as premissas fixas do Bloco A, e gerar os três arquivos de
  memória (`AUDIT.md`, `DECISIONS.md`, `PROGRESS.md`) pela primeira vez. **Se já existir
  `docs/AUDIT.md`** (gerado antes pela skill `project-discovery-42it`, ou por uma Fase 0 anterior), a
  Fase 0 fica mais leve: revisar/atualizar essa auditoria à luz da mudança agora pedida, em vez de
  auditar do zero — mas ainda gera `DECISIONS.md` e `PROGRESS.md` pela primeira vez, se ainda não
  existirem. Se o projeto for greenfield (sem código ainda), a Fase 0 vira "montar o esqueleto do
  repositório + memória" em vez de auditoria, mas o espírito é o mesmo: nada de feature ainda.
- **Fase 1 é sempre a fundação**: as peças de que tudo mais depende (autenticação, modelo de dados,
  armazenamento, uma integração crítica, o primeiro fluxo central do produto). Quebre a Fase 1 em
  subprompts (1A, 1B, 1C…) — um por peça de risco — em vez de um prompt gigante. Cada peça de risco
  vira um subprompt quando: mexe em schema, em autenticação/autorização, em dados sensíveis, em
  storage, ou em algo que, se sair errado, é caro de desfazer.
- **Fases seguintes (2, 3…) são funcionalidades construídas em cima da fundação**, na ordem de
  prioridade que o usuário indicou.
- **Qualquer coisa fora do escopo atual vira uma linha de "backlog futuro"**, citada mas não
  detalhada em prompt. Não infle o plano com fases que ninguém vai rodar tão cedo.
- Não crie subfases artificialmente: se o projeto for pequeno, a Fase 1 pode ser um único prompt.
  O critério é risco e tamanho de contexto por sessão, não "parecer mais completo".

## Passo 4 — Plan Mode vs. modo de execução, por prompt

Ao anotar cada prompt do plano, use esta regra (o texto completo de explicação do conceito já está
no template, isso aqui é só o critério de decisão):

- **Modo de execução direto**: tarefas só de leitura/documentação (como a Fase 0), ou mudanças
  contidas a um único arquivo, de baixo risco.
- **Plan Mode primeiro, depois execução**: qualquer coisa multi-arquivo, mudança de schema,
  autenticação/autorização, pagamento, storage ou controle de acesso, ou refatoração em código que
  já está em produção com usuários reais.

Anote isso como uma linha curta logo acima de cada bloco de prompt, no formato usado no template
("**Modo recomendado: ...**" seguido de uma frase de porquê).

## Passo 5 — Pontos em aberto (não pular)

Ao montar o plano, é comum notar riscos de produto que o usuário não mencionou explicitamente —
privacidade de dados, moderação de conteúdo público, custo de infraestrutura que escala com
clientes/uso, segurança de links/identificadores previsíveis, LGPD/GDPR se houver dado pessoal.
Liste esses pontos na seção final do documento, mesmo que o usuário não tenha perguntado. Não
esconda um risco identificado só para deixar o plano com aparência mais "limpa" ou definitiva.

## Passo 6 — Escolher o formato de entrega

Dois modos, não são mutuamente exclusivos — dá pra entregar os dois:

**Modo documento** (era o único modo antes desta skill ganhar o modo kit): um arquivo markdown
com o plano completo — contexto fixo, roadmap, um prompt por fase escrito no corpo do documento —
para o usuário copiar cada prompt manualmente na ferramenta agentic. Use este modo quando: a sessão
não tem como escrever arquivo nenhum (nem no repositório do usuário, nem para download); o projeto
tem só uma fase ou duas (o problema de "esquecer um prompt no meio de muitos" não existe ainda); ou
o usuário pediu explicitamente só o documento para ler ou levar para outra ferramenta que não leia
`CLAUDE.md`/`AGENTS.md`.

**Modo kit** (novo, recomendado sempre que o projeto tiver várias fases e a sessão conseguir
escrever arquivo): em vez de um prompt por fase dentro de um documento gigante, cada fase vira um
arquivo próprio em `docs/fases/`, mais um protocolo de fases no `CLAUDE.md` que faz o agente
resolver a fase a partir de uma frase natural ("próximo", "começar fase-1c") lendo o progresso
salvo — sem depender de a pessoa lembrar qual prompt vem a seguir nem de digitar o nome exato de um
comando — mais subagentes para os papéis especializados que se repetem no roadmap. Veja o Passo 7
para os detalhes de como montar isso.

Em qualquer um dos dois modos:

- Sempre gere também o documento de overview do Passo 2 (mesmo em modo kit) — ele é a referência
  humana de tudo que foi decidido; no modo kit, a seção de prompts por fase pode virar só uma lista
  com o nome de cada fase e uma frase, apontando para o arquivo de fase correspondente, em vez de
  repetir o prompt inteiro duas vezes.
- Não execute nenhum prompt de fase por conta própria nesta skill, em nenhum dos dois modos — o
  trabalho aqui é só preparar o plano (e, em modo kit, os arquivos que automatizam a mecânica de
  seguir o roadmap). Quem decide quando rodar cada fase, e quando aprovar cada PR, é o usuário —
  mas os planos e kits gerados devem instruir o agente a, ao final de cada fase, perguntar se ela
  está ok e, diante de uma resposta positiva, já fazer o merge do PR e a limpeza das branches, sem
  exigir um novo pedido do usuário para isso (formato em `references/claude_code_kit.md`).
- Depois de entregar, ofereça (sem insistir) ajustar qualquer fase, adicionar subfases, detalhar
  mais algum prompt específico, ou — se entregou só o documento — gerar o kit de arquivos a partir
  dele.

## Passo 7 — Gerar o kit de automação (modo kit)

Use `references/claude_code_kit.md` como esqueleto operacional — ele tem o formato exato de cada
arquivo. Aqui vai o raciocínio de por que cada peça existe e como decidir o que gerar.

**A ideia central**: hoje, para rodar um plano de várias fases, a pessoa precisa copiar cada prompt
manualmente, na ordem certa, sem pular nenhum — e num projeto grande é fácil perder o fio ou colar
o prompt errado. O kit resolve isso trocando "prompt dentro de um documento" por "prompt como
arquivo no repositório", e adicionando um protocolo no `CLAUDE.md` que sabe qual é a próxima fase
sem depender da memória de ninguém.

**Por que contexto e não comando nomeado**: um comando (`/fase-1c`) exige que a pessoa lembre o
nome exato, exige manter o nome idêntico em vários lugares, e só existe em ferramentas que têm
comando nomeado. O `CLAUDE.md` é a única coisa que qualquer ferramenta agentic carrega sozinha em
toda sessão nova — então é ali que mora a instrução de como resolver "próximo" ou "1c". O custo é
menos determinismo (uma instrução pode ser mal interpretada), mitigado pela confirmação de uma linha
antes de executar cada fase. Comandos nomeados continuam possíveis como atalho opcional, só se o
usuário pedir (ver o arquivo de referência).

**As quatro peças do kit, e por que cada uma existe:**

1. **Um arquivo por fase/subfase** (`docs/fases/<fase>.md` — markdown simples, sem front matter;
   nomeie `fase-0`, `fase-1a`, `fase-1b`, `fase-2`... de forma idêntica em três lugares: no roadmap
   do documento do Passo 2, no checklist do `docs/PROGRESS.md`, e no nome deste arquivo). Isto
   substitui o prompt que antes vivia só dentro do documento — não duplique o conteúdo nos dois
   lugares, o arquivo de fase é a fonte única. Cada arquivo continua seguindo o esqueleto do Passo 4
   do `references/plan_template.md` (o que ler antes, tarefas, o que não fazer, branch/commit/PR,
   atualizar memória, encerramento) — ele só é lido quando a fase é resolvida, então não pesa no
   contexto de início de sessão.

2. **`docs/PROGRESS.md` num formato de checklist padronizado** (`- [ ]` / `- [x]`, ver o formato
   exato no arquivo de referência) — isto já existia como arquivo de memória, mas agora também
   precisa ser uma fonte de dados legível por máquina, porque é dali que o protocolo descobre qual
   é a próxima fase pendente. Se o formato variar de projeto para projeto, o protocolo quebra
   silenciosamente — siga o formato à risca.

3. **Uma seção "Protocolo de fases" no `CLAUDE.md`/`AGENTS.md`** (texto pronto no arquivo de
   referência). É o que permite ao usuário abrir a ferramenta agentic e dizer algo curto como
   "próximo", "começar fase-1c" ou "vamos seguir com a fase 0 e a fase 1" em vez de colar prompt por
   prompt: o agente lê `docs/PROGRESS.md`, resolve a fase (a citada, ou só a próxima pendente se não
   citar nenhuma), confirma em uma linha qual vai executar, executa na ordem certa, e para para
   reportar entre cada uma — sem pular etapa e sem inventar uma fase que não esteja no roadmap. Ele
   **não** pula o checkpoint de Plan Mode e **não** faz merge de PR por conta própria: ao final de
   cada fase pergunta se ela está ok, e só com resposta positiva explícita do usuário faz o merge do
   PR e a limpeza das branches da fase. Evita o trabalho manual de copiar prompt, de lembrar qual
   vem a seguir, e de mergear e apagar branch na mão. O conteúdo dessa seção é praticamente fixo
   entre projetos — copie o template do arquivo de referência quase sem alteração, e mantenha-a
   curta, porque o `CLAUDE.md` é lido em toda sessão.

4. **Um subagente por papel especializado que se repete no roadmap** (`.claude/agents/<papel>.md` —
   ex.: um papel de auditoria/QA que só lê e revisa, um papel de implementação de backend, um de
   frontend). Isto é o que atende ao pedido de "iniciar um subagente sem muito contexto": por
   padrão, um subagente começa sem o histórico da conversa principal — só com `CLAUDE.md`, a
   tarefa que foi delegada a ele, e o próprio prompt do papel — e devolve um resumo para a sessão
   principal ao terminar. Não crie um subagente para um papel que aparece uma única vez no roadmap;
   o overhead de manter o arquivo não compensa. Dentro do arquivo de cada fase, aponte explicitamente
   quando delegar a um desses subagentes ("use o subagente `<papel>` para X").

**Decidindo quais subagentes criar**: releia o roadmap do Passo 3 e agrupe as tarefas por
especialidade recorrente (não por fase — um mesmo papel costuma aparecer em várias fases). Dois ou
mais aparições da mesma especialidade justificam um subagente; uma aparição isolada não.

**Outras ferramentas além do Claude Code**: se o usuário vai usar Antigravity, OpenCode ou outra,
use `references/other_tools.md` — quase todo o kit generaliza direto, porque funciona por contexto
(CLAUDE.md/AGENTS.md com o protocolo, os arquivos de memória, um arquivo por fase), e só uma parte
não tem equivalente confirmado em toda ferramenta (subagente com contexto isolado, comando nomeado
como atalho). Seja direto sobre o que não dá para replicar em vez de inventar uma convenção —
confirme a convenção da ferramenta antes de gerar qualquer arquivo num formato específico dela, não
assuma pelo que valeu para outra ferramenta.

**Ao entregar**: gere todos os arquivos do kit numa só rodada, resuma em poucas linhas o que foi
criado e onde, e diga como começar (ex.: "salve esta estrutura na raiz do repositório, abra o
Claude Code lá e diga 'começar fase-0', ou só 'próximo' se quiser que ele decida a próxima fase
sozinho"). Não execute nenhuma fase você mesmo nesta conversa.

## Qualidade — o que separa um bom plano de um genérico

- As premissas do Bloco A precisam ser específicas do produto real, não frases-clichê tipo "seja
  simples e escalável". Se você escreveria a mesma frase para qualquer produto, ela está genérica
  demais — reescreva com o detalhe real que o usuário deu.
- Cada prompt de fase precisa ter, sempre: o que ler antes de começar, o que fazer, o que
  explicitamente **não** fazer nesta fase (delimitar escopo evita que o agente invente
  funcionalidade), critério de pronto, branch/commit/PR, e instrução de atualizar a memória ao
  final.
- Se o usuário anexar um plano anterior para "aprimorar", releia-o inteiro antes de sugerir
  qualquer mudança — não repita trabalho nem contradiga decisões que já constam em
  `docs/DECISIONS.md` sem sinalizar a contradição explicitamente.
- **Específico do modo kit**: o nome de cada fase precisa ser idêntico nos três lugares onde ele
  aparece (roadmap do documento, checklist do `docs/PROGRESS.md`, nome do arquivo em
  `docs/fases/`) — é o tipo de inconsistência que não dá erro visível, só faz o agente pular ou
  repetir fase sem avisar. Cada arquivo de fase precisa incluir o bloco de encerramento (pergunta "a
  fase está ok?" + merge do PR + limpeza das branches, só após resposta positiva explícita) e ser
  autocontido (ele pode ser lido numa sessão nova, sem o resto desta conversa por perto): instrua-o
  a *ler* `docs/PROGRESS.md`/`DECISIONS.md` na hora, nunca copie o conteúdo atual deles para dentro do prompt da fase. Não crie subagente
  para papel que aparece uma vez só no roadmap, e escreva a `description` de cada subagente
  específica o bastante para não colidir com outro papel na hora de decidir para quem delegar.
