# Template — `docs/AUDIT.md`

> Esqueleto do arquivo de auditoria do projeto. Usado tanto por esta skill (quando roda antes de
> qualquer plano existir) quanto pela Fase 0 da skill `agent-orchestration-plan-42it` (quando ainda não
> há auditoria prévia e ela precisa ser gerada como parte do plano). O formato é o mesmo nos dois
> casos, de propósito — é o que permite uma alimentar a outra sem fricção nem reformatação.

```markdown
# Auditoria do Projeto — [NOME DO PROJETO]

> Gerado em [data] por [project-discovery-42it / Fase 0 do plano "[nome do plano]"]. Documento de
> estado — só leitura, não prescreve solução.

## Visão geral
[O que existe hoje: stack principal, linguagens, tamanho aproximado (nº de módulos/serviços),
como o projeto está organizado em alto nível.]

## Como funciona hoje, por área

### Autenticação / autorização
[O que existe, com base no código real, ou "não existe ainda" se for o caso.]

### Dados / armazenamento
[Modelo de dados atual, banco usado, como é feita a persistência.]

### Fluxo core do produto
[O caminho principal que o produto resolve, ponta a ponta, como está implementado hoje.]

### [Outras áreas relevantes a este projeto]
[Adicione quantas seções forem relevantes — não force as três acima se o projeto não tiver, por
exemplo, autenticação.]

## Convenções observadas
[Naming, estrutura de pastas, padrão de testes, linter/formatter configurado, convenção de commit
já em uso — mesmo que informal ou inconsistente.]

## Pontos fracos, incompletos ou inconsistentes
[Liste o que foi encontrado, com referência a arquivo/trecho quando possível. Não prescreva a
solução aqui — isso é trabalho de um plano de fases depois, não desta auditoria. Evite
generalização vaga; diga onde e por quê.]

## Decisões técnicas aparentes no código (a confirmar)
[Decisões que o código já deixa claras, mas que precisam de confirmação do usuário antes de virar
premissa fixa em `CLAUDE.md` ou entrada formal em `docs/DECISIONS.md`. Ex.: "parece multi-tenant
por instância separada — confirmar se é intencional ou acidental."]

## Perguntas em aberto
[O que não foi possível inferir do código e ainda não foi respondido.]
```
