# Guia do plugin `mogai-devops-pipeline` (v1.5.0)

Este guia explica o que o plugin faz, como instalar e como usar cada recurso, com exemplos.

> **Resumo:** o plugin é uma esteira de entrega guiada por especificação. Você descreve
> o que quer, e agentes especializados levam isso de **requisito → clarificação →
> especificação → arquitetura → implementação → infra → testes → documentação**. A
> esteira para nos pontos de decisão humana e dá preferência ao reuso do **Atlas SDK**
> em vez de implementação nova.

---

## Sumário

1. [Visão geral](#1-visão-geral)
2. [Pré-requisitos e instalação](#2-pré-requisitos-e-instalação)
3. [Comandos](#3-comandos)
4. [Agentes](#4-agentes)
5. [Hooks (guardrails automáticos)](#5-hooks-guardrails-automáticos)
6. [Integração com Azure DevOps (MCP)](#6-integração-com-azure-devops-mcp)
7. [Skills de referência](#7-skills-de-referência)
8. [Artefatos gerados](#8-artefatos-gerados)
9. [Briefing: como escrever o pedido](#9-briefing-como-escrever-o-pedido)
10. [Exemplos práticos](#10-exemplos-práticos)
11. [Atlas SDK](#11-atlas-sdk)
12. [O que a esteira NÃO faz sozinha](#12-o-que-a-esteira-não-faz-sozinha)
13. [Solução de problemas](#13-solução-de-problemas)
14. [Perguntas frequentes](#14-perguntas-frequentes)

---

## 1. Visão geral

| Componente | Quantidade | Para que serve |
|---|---|---|
| Comandos | 4 | Portas de entrada: `ship-feature`, `evolve-feature`, `clarify`, `atlas-audit` |
| Agentes | 11 | Especialistas acionados pelos comandos, ou chamados isoladamente |
| Hooks | 3 | Guardrails automáticos: bloqueio de segredos, formatação, testes obrigatórios |
| Servidor MCP | 1 | Acesso ao Azure DevOps (work items, repositórios, PRs, iterações) |
| Skills | 2 | Referências de metodologia: Spec-Driven Development e DDD |

### Princípios do plugin

- **A spec é o artefato central:** o código sai da especificação, não o contrário.
- **Reusar o Atlas primeiro:** antes de implementar algo do zero, verifique se um módulo do Atlas SDK já resolve.
- **Nenhuma ambiguidade passa adiante:** decisões de negócio em aberto são levadas ao humano no gate `/clarify`.
- **Humano no controle:** nenhum merge, deploy ou criação de work item acontece sem aprovação explícita.
- **Segurança fail-closed:** se um hook de segurança não consegue avaliar uma ação, ele bloqueia.

### Fluxo — projeto novo (`/ship-feature`)

```
Pedido/briefing
   │
   ▼
requirements-analyst ──► backlog.md
   │
   ▼
/clarify (GATE) ◄──► você responde às decisões pendentes
   │
   ▼
spec-writer ──► spec.md (com reuso do Atlas)
   │
   ▼
devops-architect ──► arch.md
   │
   ├──► backend-implementer  ┐ (em paralelo)
   └──► frontend-implementer ┘
   │
   ▼
devops-engineer ──► IaC + CI/CD   (PARA antes de apply/deploy)
   │
   ▼
test-runner ──► testes (hook impede encerrar com falhas)
   │
   ▼
doc-writer ──► docs/TECHNICAL.md + docs/USER-GUIDE.md
   │
   ▼
RESUMO PARA REVISÃO HUMANA
```

### Fluxo — projeto existente (`/evolve-feature`)

O fluxo é o mesmo, mas com **`codebase-discovery` primeiro** (gera `discovery.md`) e
todas as etapas seguintes em **modo delta**: só o que muda, preservando os contratos e
as convenções já existentes e protegendo contra regressão.

---

## 2. Pré-requisitos e instalação

### 2.1 Ferramentas na máquina

| Ferramenta | Para quê | Instalação (Windows) |
|---|---|---|
| Claude Code (CLI) | Rodar o plugin (não funciona no claude.ai web) | claude.com/claude-code |
| Node.js | `npx` sobe o MCP do Azure DevOps; fallback dos hooks | `winget install OpenJS.NodeJS.LTS` |
| Azure CLI | Autenticação no Azure DevOps | `winget install Microsoft.AzureCLI` |
| jq (recomendado) | Parser JSON dos hooks de segurança | `winget install jqlang.jq` |
| git | Branches e PRs | git-scm.com |
| .NET SDK / npm do projeto | Compilar e testar o projeto-alvo | Versão fixada no projeto |

> Sem `jq` **e** sem `node`, os hooks de segurança bloqueiam toda escrita de arquivo. É de propósito (fail-closed).

### 2.2 Instalar o plugin (self-service)

Depois de configurar a Deploy Key de leitura (ver `docs/CONFIGURACAO-AMBIENTE-DEV.md` do plugin):

```bash
claude plugin marketplace add git@github.com-mogai-devops-pipeline:mogaigit/mogai-devops-pipeline.git
claude plugin install mogai-devops-pipeline@mogai-devops-pipeline
```

Para testar uma cópia local sem instalar:

```bash
claude plugin validate ./mogai-devops-pipeline
claude --plugin-dir /caminho/para/mogai-devops-pipeline
```

### 2.3 Autenticar e verificar

```bash
az login
```

Dentro do Claude Code:

```
/mcp              # deve listar o azure-devops como conectado
/reload-plugins   # recarrega depois de atualizar o plugin
```

### 2.4 Atualizações

O plugin é versionado pelo campo `version` do `plugin.json`. Quem já tem o plugin instalado
recebe a atualização na próxima sessão ou ao rodar `/reload-plugins`.

---

## 3. Comandos

Todos os comandos têm o prefixo `/mogai-devops-pipeline:`.

### 3.1 `/mogai-devops-pipeline:ship-feature` — projeto novo

**Quando usar:** sistema ou módulo criado do zero (greenfield).

**Sintaxe:**
```
/mogai-devops-pipeline:ship-feature "<descrição ou caminho do briefing>"
```

**Etapas:** requisitos → clarificação → especificação → arquitetura → implementação
(backend e frontend em paralelo) → infra → testes → documentação → resumo.

**Saída final:** histórias entregues, requisitos EARS atendidos, módulos Atlas reusados,
decisões tomadas no `/clarify`, estado dos testes e documentação gerada.

### 3.2 `/mogai-devops-pipeline:evolve-feature` — projeto existente

**Quando usar:** manutenção ou evolução de um sistema que já está em produção, como este repositório.

**Sintaxe:**
```
/mogai-devops-pipeline:evolve-feature "<mudança desejada>" "<repositório/projeto alvo>"
```
Se o segundo argumento ficar vazio, o orquestrador pergunta qual é o projeto alvo antes de seguir.

**Etapas e paradas obrigatórias:**

| # | Etapa | Agente | Para quando… |
|---|---|---|---|
| 1 | Reconhecimento | `codebase-discovery` | O projeto não tem testes ou CLAUDE.md próprio (você decide se segue) |
| 2 | Requisitos | `requirements-analyst` | — |
| 3 | Clarificação | `/clarify` | Sempre, se houver decisões pendentes |
| 4 | Spec delta | `spec-writer` | Há breaking change (você decide o versionamento) |
| 5 | Análise de impacto + rollback | `devops-architect` | — |
| 6 | Implementação | `backend-` e `frontend-implementer` | — |
| 7 | Infra incremental | `devops-engineer` | Sempre, antes de apply/deploy |
| 8 | Testes + regressão | `test-runner` | A linha de base regrediu |
| 9 | Docs delta | `doc-writer` | — |

**Saída final:** o que mudou e o que ficou intacto, breaking changes e a decisão tomada sobre
eles, suíte de testes (linha de base × atual), plano de rollback e seções de documentação atualizadas.

### 3.3 `/mogai-devops-pipeline:clarify` — gate de desambiguação

**Quando usar:** roda automaticamente dentro dos dois pipelines, mas também pode ser chamado sozinho.

**Sintaxe:**
```
/mogai-devops-pipeline:clarify              # usa backlog.md
/mogai-devops-pipeline:clarify outro-backlog.md
```

**Como funciona:**
1. Lê as "Decisões pendentes" e procura termos vagos ("rápido", "vários", "amigável").
2. Monta **perguntas fechadas**, com opções concretas.
3. **Para e espera** suas respostas.
4. Converte cada resposta em requisito EARS definitivo e limpa as pendências.
5. Marca o backlog como **PRONTO PARA SPEC**.

**Dica:** responda de forma curta e direta ("24h", "sim", "só administradores").

### 3.4 `/mogai-devops-pipeline:atlas-audit` — auditoria do Atlas SDK

**Quando usar:** antes de padronizar o uso de um módulo do Atlas. Fluxo **só de leitura**.

**Sintaxe:**
```
/mogai-devops-pipeline:atlas-audit                          # todos os módulos do ATLAS.md
/mogai-devops-pipeline:atlas-audit Atlas.Net.Sdk.Cache      # um módulo
```

**Saída:**
- Um laudo por módulo, `atlas-audit-<modulo>.md`, com veredito **PRONTO / PRONTO COM RESSALVAS / NÃO RECOMENDADO**.
- Um panorama consolidado: riscos críticos e altos ordenados, e quais módulos podem ser padronizados já, quais precisam de contribuição antes e quais dependem de decisão.

---

## 4. Agentes

Os agentes podem ser acionados pelos comandos **ou isoladamente**, pedindo em linguagem natural:
> "Acione o `devops-architect` com este spec.md"

| Agente | Modelo | Acesso | Papel | Artefato |
|---|---|---|---|---|
| `codebase-discovery` | sonnet | **Só leitura** | Levanta as convenções reais, contratos publicados, módulos Atlas em uso e a linha de base de testes | `discovery.md` |
| `requirements-analyst` | sonnet | Lê e escreve work items* | Histórias de usuário, EARS + Gherkin, MoSCoW, "Decisões pendentes" | `backlog.md` |
| `spec-writer` | sonnet | Leitura + Azure DevOps | Spec técnica (contrato de API, modelo de dados, reuso do Atlas, breaking changes) | `spec.md` |
| `devops-architect` | sonnet | Leitura | Topologia, deploy, CI/CD, observabilidade; no modo delta, análise de impacto e rollback | `arch.md` |
| `backend-implementer` | sonnet | Escrita | Implementa a API .NET compondo com o Atlas backend | Código |
| `frontend-implementer` | sonnet | Escrita | Implementa a UI compondo com o `@mogai/atlas-ui` | Código |
| `devops-engineer` | sonnet | Escrita | IaC, pipelines, ambientes e secrets; **nunca faz apply ou deploy** | IaC / YAML |
| `test-runner` | haiku | Escrita | Testes a partir do Gherkin (unitários, integração HTTP real, caminhos de erro) | Testes |
| `doc-writer` | haiku | Escrita | Documentação técnica e manual do usuário a partir do que **foi de fato** entregue | `docs/*.md` |
| `atlas-auditor` | sonnet | **Só leitura** | Laudo de risco de um módulo do Atlas | `atlas-audit-*.md` |
| `atlas-contributor` | sonnet | Escrita + PR | Evolui o Atlas por branch + PR + work item; **nunca faz merge** | Branch + PR |

\* Só com a sua aprovação. Ele sempre escreve primeiro no `backlog.md`.

### Comportamento em modo delta (quando existe `discovery.md`)

- **spec-writer:** os contratos são aditivos por padrão; qualquer mudança em contrato existente é marcada como breaking change.
- **implementers:** seguem as convenções **reais** do projeto, mesmo quando divergem do padrão do plugin. A divergência é sinalizada, mas não corrigida por conta própria.
- **devops-architect:** analisa o impacto sobre a infra existente e define rollback para cada mudança.
- **test-runner:** roda a suíte completa e compara com a linha de base.
- **doc-writer:** atualiza só as seções afetadas.

---

## 5. Hooks (guardrails automáticos)

| Hook | Evento | O que faz | Se não consegue avaliar |
|---|---|---|---|
| `block-secret.sh` | Antes de cada Write/Edit | Bloqueia gravação de conteúdo que pareça credencial (token, senha, chave) | **Bloqueia** (fail-closed) |
| `format-on-save.sh` | Depois de cada Write/Edit | Formata o arquivo (prettier/black) | Pula a formatação (fail-open) |
| `require-tests-pass.sh` | Fim do `test-runner` | Impede o agente de encerrar enquanto houver teste falhando | **Bloqueia** (fail-closed) |

**Na prática:**
- Se uma escrita for bloqueada por "secret aparente", mova o valor para variável de ambiente, user-secrets ou Key Vault.
- O `format-on-save` usa prettier por padrão. Se o projeto tiver `.prettierrc`, ele é respeitado.
- O comando de testes padrão do hook é `npm test`.

---

## 6. Integração com Azure DevOps (MCP)

O plugin sobe o servidor `@azure-devops/mcp` para a organização **`Mogai`**, com os domínios
`core`, `work`, `work-items` e `repositories`.

**O que fica disponível:**

| Área | Exemplos de operação |
|---|---|
| Core | Listar projetos, times, identidades |
| Work items | Ler, criar e atualizar itens; consultas WIQL; comentários; links; anexos; backlog |
| Work | Iterações, capacidade do time |
| Repositórios | Repos, branches, arquivos, commits, pull requests e threads de comentário |

**Exemplos de pedidos:**
```
Leia a Feature 1234 do Azure DevOps e resuma os critérios de aceite.
Liste os PRs abertos no repositório LogSolutions-web.
Crie uma User Story para cada história do backlog.md (mostre antes de criar).
Quais work items estão na iteração atual do meu time?
```

> Toda escrita no Azure DevOps (criar work item, abrir PR, comentar) pede confirmação.

---

## 7. Skills de referência

| Skill | Conteúdo | Quando consultar |
|---|---|---|
| `spec-driven-development` | Notação EARS, Gherkin, templates de briefing e de `spec.md`, técnica de clarificação | Para escrever requisitos, briefings ou specs |
| `domain-driven-design` | Linguagem ubíqua, bounded contexts, agregados, quando DDD tático é exagero | Para modelar o domínio ou definir fronteiras de serviço |

Exemplo:
```
Use a skill spec-driven-development e reescreva este requisito em EARS.
```

### EARS em 30 segundos

| Padrão | Forma | Exemplo |
|---|---|---|
| Ubíquo | O sistema deve… | O sistema deve registrar a data de cada changelog. |
| Evento | Quando \<gatilho\>, o sistema deve… | Quando o usuário salvar um menu, o sistema deve validar o ícone. |
| Estado | Enquanto \<estado\>, o sistema deve… | Enquanto a sessão estiver expirada, o sistema deve redirecionar ao login. |
| Indesejado | Se \<condição\>, então o sistema deve… | Se a rota `/me` falhar, então o sistema deve exibir mensagem de erro. |
| Opcional | Onde \<feature\>, o sistema deve… | Onde houver multitenancy, o sistema deve filtrar por tenant. |

---

## 8. Artefatos gerados

| Arquivo | Gerado por | Conteúdo |
|---|---|---|
| `discovery.md` | codebase-discovery | Retrato do projeto existente e linha de base de testes |
| `backlog.md` | requirements-analyst / clarify | Histórias, EARS, Gherkin, MoSCoW, decisões |
| `spec.md` | spec-writer | Sumário, domínio, contrato de API, dados, estado, idempotência, reuso do Atlas |
| `arch.md` | devops-architect | Topologia, deploy, CI/CD, observabilidade, impacto e rollback |
| `docs/TECHNICAL.md` | doc-writer | Documentação técnica |
| `docs/USER-GUIDE.md` | doc-writer | Manual do usuário |
| `docs/adr/*.md` | (convenção) | Registro de decisões de arquitetura |
| `atlas-audit-*.md` | atlas-auditor | Laudos dos módulos do Atlas |

> **Dica:** revise cada artefato antes de liberar a próxima etapa. Corrigir no `spec.md` sai muito mais barato do que corrigir no código.

---

## 9. Briefing: como escrever o pedido

Um bom briefing reduz as perguntas do `/clarify`. Os formatos de projeto novo e de evolução são diferentes; não misture os dois.

### 9.1 Projeto novo (`/ship-feature`)

```markdown
# Briefing — <nome>

## 1. Contexto de negócio
Situação atual, problema, quem é afetado.

## 2. Objetivo
O que o sistema deve permitir (1–2 frases).

## 3. Stakeholders / Personas
Quem usa e quem decide.

## 4. Escopo do MVP
- Item que entra
- Item que entra

## 5. Fora do escopo
- Item que NÃO entra

## 6. Restrições
Stack, ambiente, regulação, regras de negócio invioláveis.

## 7. Referências
Epic/Feature no Azure DevOps (se houver).
```

### 9.2 Evolução (`/evolve-feature`)

```markdown
# Briefing de evolução — <mudança>

## 1. Sistema alvo
LogSolutions-web

## 2. Mudança solicitada
Descrição objetiva e pequena.

## 3. Motivação
Por que agora.

## 4. Comportamento esperado (antes → depois)
Antes: ...
Depois: ...

## 5. O que NÃO pode quebrar
Contratos e invariantes a preservar. Se a quebra for intencional, diga isso aqui.

## 6. Restrições adicionais
Janela de deploy, dependência de outro time etc.

## 7. Referências
Work item relacionado.
```

---

## 10. Exemplos práticos

> Os exemplos abaixo são ilustrativos e usam funcionalidades deste repositório como inspiração.

### Exemplo 1 — Evolução simples a partir de uma frase

```
/mogai-devops-pipeline:evolve-feature "Adicionar filtro por período (data inicial/final) na listagem de changelogs" "LogSolutions-web"
```

O que acontece:
1. O `codebase-discovery` mapeia como as telas de listagem são feitas hoje (componentes, serviços, padrão de filtros) e roda os testes existentes para registrar a linha de base.
2. O `requirements-analyst` escreve a história e deixa pendências, por exemplo: *"O filtro é inclusivo nas datas-limite?"* e *"Qual o período padrão?"*.
3. O `/clarify` pergunta, você responde: *"Inclusivo; padrão últimos 30 dias."*
4. O `spec-writer` define, por exemplo, os parâmetros opcionais `dataInicio` e `dataFim` (contrato aditivo, sem breaking change).
5. O `frontend-implementer` cria o filtro usando o mesmo padrão visual das outras telas.
6. O `test-runner` roda a suíte completa e confirma que não houve regressão.
7. O `doc-writer` atualiza só a seção de changelogs no `USER-GUIDE.md`.
8. Você recebe o resumo e decide sobre commit, PR e deploy.

### Exemplo 2 — Evolução a partir de um briefing em arquivo

```
/mogai-devops-pipeline:evolve-feature "Ver briefing em docs/briefings/icon-picker-busca.md" "LogSolutions-web"
```

### Exemplo 3 — Evolução a partir de um work item do Azure DevOps

```
/mogai-devops-pipeline:evolve-feature "Implementar a User Story 4567 do Azure DevOps" "LogSolutions-web"
```
O `requirements-analyst` lê o work item primeiro e aproveita o contexto que já está registrado.

### Exemplo 4 — Projeto novo

```
/mogai-devops-pipeline:ship-feature "Portal interno para solicitação de veículos da frota, com aprovação do gestor e notificação por e-mail"
```

### Exemplo 5 — Rodar só o gate de clarificação

```
/mogai-devops-pipeline:clarify backlog.md
```
Exemplo de pergunta gerada:
> **P3.** O cancelamento de uma solicitação é permitido até quando?
> (a) A qualquer momento antes da aprovação · (b) Até 24h antes do uso · (c) Nunca, só rejeição pelo gestor

### Exemplo 6 — Refazer só uma etapa

```
Acione o devops-architect com o spec.md atual e o discovery.md, gerando apenas a análise de impacto e o plano de rollback.
```
```
Acione o test-runner para gerar testes dos critérios Gherkin da história H2 do backlog.md.
```
```
Acione o doc-writer em modo delta para atualizar a documentação da rota /me.
```

### Exemplo 7 — Só reconhecimento do projeto (sem mudar nada)

```
Acione o codebase-discovery neste repositório e gere o discovery.md.
```
Útil para onboarding ou antes de estimar uma demanda.

### Exemplo 8 — Auditar e melhorar o Atlas SDK

```
/mogai-devops-pipeline:atlas-audit Atlas.Net.Sdk.Security.Jwt
```
Depois, com o laudo em mãos:
```
Acione o atlas-contributor para adicionar testes ao Atlas.Net.Sdk.Security.Jwt conforme o laudo atlas-audit-Atlas.Net.Sdk.Security.Jwt.md, item "Testes: sem cobertura de expiração de token".
```
O agente cria a branch, segue TDD, roda a suíte, vincula um work item e abre o PR. **Não faz merge.** Mudanças no módulo JWT exigem revisão de um sênior.

### Exemplo 9 — Operações avulsas no Azure DevOps

```
Liste minhas tarefas ativas na sprint atual.
Mostre os comentários do PR 712.
Crie uma Task filha da User Story 4567 chamada "Ajustar validação do icon-picker".
```

---

## 11. Atlas SDK

O Atlas é o SDK interno da Mogai, distribuído por um **feed privado do Azure Artifacts**.

### Backend — monorepo `Atlassdk-Dotnet` (23 pacotes NuGet)

| Área | Pacotes |
|---|---|
| Núcleo / ASP.NET | `Core`, `AspNet`, `Plugin` |
| Dados | `Data`, `Data.Common`, `Data.Providers.{Oracle,PostgreSQL,SQLServer}`, `Data.Tenant.{Oracle,SQL}` |
| Logging | `Logger`, `Logger.Mvc`, `Logger.{Oracle,PostgreSQL,SQLServer}` |
| Segurança | `Security.Jwt`, `GoogleAuth` |
| Infra | `Cache`, `Message`, `Queue`, `Audit` |
| Licenciamento | `License`, `License.UI` |

Todos com o prefixo `Atlas.Net.Sdk.`. O que cada um cobre foi inferido do nome; confirme na documentação antes de fixar numa spec.

### Frontend — `AtlasSDK-UI` (`@mogai/atlas-ui`)

- Angular 20 + NG-ZORRO 20, com 38 componentes no design system.
- A branch de referência é `main`.

### Fora de escopo

- `AtlasSDK-JS` (NodeJS/NestJS): desconsiderado pelo pipeline.

> O Atlas veio de uma aquisição e a qualidade dele está sob avaliação. Antes de padronizar um módulo, rode `/atlas-audit`.

---

## 12. O que a esteira NÃO faz sozinha

- ❌ Merge de PR
- ❌ Deploy ou apply de infraestrutura real
- ❌ Criar ou atualizar work items sem aprovação
- ❌ Decidir regra de negócio ambígua
- ❌ Usar tags de imagem mutáveis (`latest`)
- ❌ Gravar credenciais em arquivo

Cada uma dessas ações exige um pedido explícito seu, feito à parte.

---

## 13. Solução de problemas

| Sintoma | Causa provável | Solução |
|---|---|---|
| `/mcp` não mostra o `azure-devops` conectado | Falta `az login` ou Node.js | Rode `az login`, confira `node -v` e depois `/reload-plugins` |
| Toda escrita de arquivo é bloqueada | `jq` e `node` ausentes (hook fail-closed) | Instale o `jq` (`winget install jqlang.jq`) |
| Escrita bloqueada por "secret aparente" | Conteúdo parece credencial | Use variável de ambiente, user-secrets ou Key Vault |
| `test-runner` não termina | Hook exige testes passando | Corrija os testes ou o código. Não é bug. |
| Agente não consegue acessar o Azure DevOps | Possível divergência entre os nomes das ferramentas MCP declaradas nos agentes e os nomes carregados pelo plugin | Verifique e registre no `KNOWN-ISSUES.md` do plugin |
| Plugin não aparece após atualizar | Cache da sessão | `/reload-plugins` ou reinicie o Claude Code |
| Formatação inesperada nos arquivos | `format-on-save` com prettier | Ajuste o `.prettierrc` do projeto |

---

## 14. Perguntas frequentes

**Preciso rodar a esteira inteira toda vez?**
Não. Use `/clarify` sozinho ou acione um agente específico para refazer uma etapa.

**Posso pular o `/clarify`?**
Não é recomendado. Ele registra as decisões no `backlog.md`, o que deixa tudo rastreável. Se você já sabe as respostas, responda rápido.

**Qual comando uso neste repositório?**
`/evolve-feature`, porque o sistema já existe e está em produção.

**E se a spec disser que um módulo do Atlas cobre algo que não cobre?**
O agente deveria sinalizar "verificar com o time do Atlas". Se não sinalizar, é bug do agente: registre no `known-issues.md` do plugin.

**Os artefatos (`backlog.md`, `spec.md`...) devem ir para o git?**
Depende da convenção do time. São úteis para rastreabilidade, mas combine com a equipe antes de commitar.

**Onde fica a documentação original do plugin?**
`~/.claude/plugins/cache/mogai-devops-pipeline/mogai-devops-pipeline/1.5.0/`. Principais arquivos: `README.md`,
`docs/GUIA-DE-USO.md`, `docs/GUIA-TREINAMENTO.md`, `docs/AZURE-DEVOPS.md`, `ATLAS.md`, `CLAUDE.md` e `KNOWN-ISSUES.md`.
