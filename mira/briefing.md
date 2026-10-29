# Briefing: Atlas SDK e plugin Mogai no fluxo DevOps
**Fontes:** referências do deck (texto + imagens) em `decks/2026-10-26 mogai-devops-sdk/references/`
- `Apresentação SDK.pptx`: 12 slides, todos imagem (sem texto editável nem notas). Os slides 10 e 11 são a mesma imagem.
- `GUIA-MOGAI-DEVOPS-PIPELINE.md`: guia do plugin `mogai-devops-pipeline` v1.5.0 (562 linhas).
- `teste-claude/plugin-mogai-01..17.png`: capturas de uma sessão real no Claude Code.
**Data da extração:** 2026-10-26

## Essência em uma frase
O Atlas SDK (backend .NET) e o Atlas UI (Angular) são a base reutilizável da Mogai, e o plugin `mogai-devops-pipeline` para o Claude Code transforma um pedido em entrega guiada por especificação, reusando o Atlas, parando nos pontos de decisão humana e integrando com o Azure DevOps.

Público: time de desenvolvimento e liderança técnica da Mogai. Objetivo: entender o SDK, preparar o ambiente e usar o plugin no dia a dia.

---

## Parte A. Atlas SDK e Atlas UI (fonte: PPTX + guia §11)

### A1. Capa do ecossistema (PPTX s1)
"Atlas SDK / Atlas UI: componentes e serviços reutilizáveis para construir aplicações corporativas modernas, seguras e escaláveis." Pilares: Atlas SDK (infra .NET), Atlas UI (componentes Angular), Segurança (JWT, RBAC, multi-tenant, auditoria), Integração (mensageria, logging), Mais produtividade.

### A2. Atlas SDK (PPTX s2)
"Biblioteca modular para .NET 10, com infraestrutura, segurança, integração e boas práticas."
- `Atlas.Net.Sdk.Core`: núcleo, abstrações e configurações base.
- `Atlas.Net.Sdk.Audit`: auditoria e rastreabilidade.
- `Atlas.Net.Sdk.Data`: persistência (EF Core, Oracle/SQL Server, Repositories/Query, Specification Query).
- Provedores: `Data.Providers.Oracle`, `Data.Providers.PostgreSQL`, `Data.Providers.SQLServer`, `Data.Common`.
- Especializados: `Security.Jwt`, `GoogleAuth`, `Message` (e-mail), `Queue` (RabbitMQ), `Logger` (logging, Elmah), `AspNet`, `Notification`, `Tenant`.
- Atributos: modular, seguro, escalável, integrado.

Guia §11: monorepo `Atlassdk-Dotnet`, **23 pacotes NuGet** no **feed privado do Azure Artifacts**, prefixo `Atlas.Net.Sdk.`. Áreas: Núcleo/ASP.NET (`Core`, `AspNet`, `Plugin`), Dados (`Data`, `Data.Common`, `Data.Providers.{Oracle,PostgreSQL,SQLServer}`, `Data.Tenant.{Oracle,SQL}`), Logging (`Logger`, `Logger.Mvc`, `Logger.{Oracle,PostgreSQL,SQLServer}`), Segurança (`Security.Jwt`, `GoogleAuth`), Infra (`Cache`, `Message`, `Queue`, `Audit`), Licenciamento (`License`, `License.UI`).
> O Atlas veio de uma aquisição e a qualidade dele está sob avaliação. Antes de padronizar um módulo, rode `/atlas-audit`.

### A3. Atlas UI (PPTX s3 + guia)
- Pacote `@mogai/atlas-ui`, versão 1.0.37 (PPTX), Angular 20 standalone, sobre NG-ZORRO, tema baseado no Falcon (Bootstrap 5.3 + FontAwesome), SimpleBar.
- Guia: Angular 20 + NG-ZORRO 20, **38 componentes**, branch de referência `main`. `AtlasSDK-JS` (NestJS) está fora do escopo do pipeline.
- Categorias: Formulários, Tabelas e Dados, Layout e Navegação, Visualização de Dados, Interação e Feedback, Especializados (Audit Timeline, Train Card, Capacity Progress).
- Benefícios: acelera o desenvolvimento, padroniza a experiência, integra com Angular Forms (Reactive e Template), Standalone + Signals, pronto para o ambiente corporativo.
- Exemplo real de uso (PPTX):
```html
<at-table
  [data]="usuarios"
  [columns]="colunas"
  [loading]="carregando"
  (onPageChange)="carregarDados($event)">
</at-table>
```

### A4. Atlas UI em ação (PPTX s4 a s7, GIFs)
- s4: Storybook, componente **Autocomplete** (busca, inputs como `placeholder`, `hasMore`, `isLoading`; outputs `loadMore`, `searchTermChange`, `optionSelected`, `clear`).
- s5: tela **Atendimentos** do Port Solutions (filtros, select, date range picker, tabela paginada).
- s6: Storybook, **CapacityProgressCard** (drag and drop entre cards, `capacity`, `dropValidationState`).
- s7: **Editor de Despacho Ferroviário** do Port Solutions (vagões, placas, resumo da composição, arrastar placas).
Os GIFs são pesados (6 a 52 MB). No deck, convém converter para vídeo leve.

### A5. Arquitetura PMS (PPTX s8)
Usuário/Cliente → Gateway (autenticação, autorização, rate limiting) → APIs (External Integration API, PMS API, IAM API) ↔ Services (Integration, PMS, IAM) → Mensageria (Queue Service + MinIO) ↔ RabbitMQ (filas 1..N). Camada transversal Atlas SDK (Core, Data, Security.Jwt, Queue, Audit, Tenant). Camada de dados: Outbound IAM e Outbound PMS (Oracle/SQL Server), Oracle principal do PMS.

### A6. Atlas + IA (PPTX s9)
Desenvolvedor define o objetivo e revisa ↔ Claude Code (entende o contexto, consulta a documentação Atlas, aplica os padrões, gera e refatora código) ↔ **Atlas Knowledge**: `CLAUDE.md`/`AGENTS.md` (regras, padrões, diretrizes de uso do Atlas), Documentação Atlas (spec.md por pacote, patterns, recipes, exemplos, ADRs), Padrões de Projeto. Ganhos: menos dúvidas, decisões mais rápidas, código alinhado, menos retrabalho.

### A7. Esteira de desenvolvimento com Claude Code e Azure (PPTX s10/s11, visão)
8 etapas: 1 Planejamento (Azure Boards) → 2 Análise e Implementação (Claude Code) → 3 Controle de Código (Azure Repos) → 4 Revisão Humana (Azure DevOps) → 5 CI/CD e Segurança (Azure Pipelines, SAST/DAST) → 6 Homologação (staging) → 7 Produção (Azure Container Apps ou App Service) → 8 Observabilidade (Azure Monitor, OpenTelemetry). Permissões: acesso mínimo, branch policies, segredos no Key Vault, ambientes isolados, auditoria.
> Atenção: esta é a visão de ponta a ponta. O plugin cobre as etapas 1 a 4 e prepara IaC/CI/CD, mas **não faz deploy nem apply** (guia §12).

### A8. Organização de docs/ (PPTX s12)
`projeto/{src,tests,docs/{features/001-autenticacao,002-faturamento},bugs/{BUG-001-erro-calculo,...},architecture,decisions},CLAUDE.md,README.md}`. Por feature: `README.md` (contexto), `requirements.md` (detalhamento), `tasks.md` (checklist que o Claude Code atualiza). Identificadores estáveis (FEAT-001, BUG-001) para PRs.

---

## Parte B. Plugin `mogai-devops-pipeline` v1.5.0 (fonte: guia)

### B1. Visão geral
4 comandos (`ship-feature`, `evolve-feature`, `clarify`, `atlas-audit`), 11 agentes, 3 hooks, 1 servidor MCP (Azure DevOps), 2 skills (Spec-Driven Development, DDD).
Princípios: a spec é o artefato central; reusar o Atlas primeiro; nenhuma ambiguidade passa adiante; humano no controle; segurança fail-closed.

### B2. Preparação do ambiente
| Ferramenta | Para quê | Instalação (Windows) |
|---|---|---|
| Claude Code (CLI) | Rodar o plugin (não funciona no claude.ai web) | claude.com/claude-code |
| Node.js | `npx` sobe o MCP do Azure DevOps; fallback dos hooks | `winget install OpenJS.NodeJS.LTS` |
| Azure CLI | Autenticação no Azure DevOps | `winget install Microsoft.AzureCLI` |
| jq (recomendado) | Parser JSON dos hooks | `winget install jqlang.jq` |
| git | Branches e PRs | git-scm.com |
| .NET SDK / npm do projeto | Compilar e testar | Versão fixada no projeto |
Sem `jq` **e** sem `node`, os hooks bloqueiam toda escrita (fail-closed, de propósito).

Instalação (depois da Deploy Key de leitura, ver `docs/CONFIGURACAO-AMBIENTE-DEV.md` do plugin):
```bash
claude plugin marketplace add git@github.com-mogai-devops-pipeline:mogaigit/mogai-devops-pipeline.git
claude plugin install mogai-devops-pipeline@mogai-devops-pipeline
```
Cópia local: `claude plugin validate ./mogai-devops-pipeline` e `claude --plugin-dir /caminho/para/mogai-devops-pipeline`.
Autenticar: `az login`. No Claude Code: `/mcp` (deve listar `azure-devops` conectado) e `/reload-plugins`.
Atualização pelo campo `version` do `plugin.json`, na próxima sessão ou com `/reload-plugins`.

### B3. Comandos (prefixo `/mogai-devops-pipeline:`)
- `ship-feature "<descrição ou caminho do briefing>"`: greenfield.
- `evolve-feature "<mudança desejada>" "<repositório/projeto alvo>"`: sistema existente, `codebase-discovery` primeiro e tudo em modo delta. Tabela de 9 etapas com paradas (reconhecimento, requisitos, clarificação, spec delta, impacto + rollback, implementação, infra incremental, testes + regressão, docs delta).
- `clarify [backlog]`: lê decisões pendentes e termos vagos, faz perguntas fechadas, para e espera, converte em EARS, marca **PRONTO PARA SPEC**.
- `atlas-audit [módulo]`: só leitura, laudo `atlas-audit-<modulo>.md` com veredito PRONTO / PRONTO COM RESSALVAS / NÃO RECOMENDADO.

### B4. Fluxo `/ship-feature`
Pedido → requirements-analyst (`backlog.md`) → /clarify (GATE) → spec-writer (`spec.md`, com reuso do Atlas) → devops-architect (`arch.md`) → backend-implementer ‖ frontend-implementer → devops-engineer (IaC + CI/CD, para antes de apply/deploy) → test-runner (hook impede encerrar com falhas) → doc-writer (`docs/TECHNICAL.md`, `docs/USER-GUIDE.md`) → resumo para revisão humana.

### B5. Agentes (11)
codebase-discovery (sonnet, só leitura, `discovery.md`), requirements-analyst (sonnet, `backlog.md`), spec-writer (sonnet, `spec.md`), devops-architect (sonnet, `arch.md`), backend-implementer (sonnet, API .NET com Atlas), frontend-implementer (sonnet, UI com `@mogai/atlas-ui`), devops-engineer (sonnet, IaC/YAML, nunca apply/deploy), test-runner (haiku, testes do Gherkin), doc-writer (haiku, `docs/*.md`), atlas-auditor (sonnet, só leitura), atlas-contributor (sonnet, branch + PR, nunca merge).

### B6. Hooks
`block-secret.sh` (antes de Write/Edit, bloqueia credencial, fail-closed), `format-on-save.sh` (depois, prettier/black, fail-open), `require-tests-pass.sh` (fim do test-runner, fail-closed; comando padrão `npm test`).

### B7. Azure DevOps (MCP)
`@azure-devops/mcp`, organização **Mogai**, domínios `core`, `work`, `work-items`, `repositories`. Toda escrita pede confirmação. Exemplos de pedido do guia: "Leia a Feature 1234 do Azure DevOps e resuma os critérios de aceite." etc.

### B8. Skills, EARS e artefatos
EARS: Ubíquo, Evento, Estado, Indesejado, Opcional (com exemplos do guia). Artefatos: `discovery.md`, `backlog.md`, `spec.md`, `arch.md`, `docs/TECHNICAL.md`, `docs/USER-GUIDE.md`, `docs/adr/*.md`, `atlas-audit-*.md`. Dica: corrigir no `spec.md` sai mais barato que no código.

### B9. Briefing de pedido
Modelos 9.1 (projeto novo, 7 seções) e 9.2 (evolução: sistema alvo, mudança, motivação, antes → depois, o que não pode quebrar, restrições, referências).

### B10. Exemplos práticos (comandos reais do guia)
```
/mogai-devops-pipeline:evolve-feature "Adicionar filtro por período (data inicial/final) na listagem de changelogs" "LogSolutions-web"
/mogai-devops-pipeline:evolve-feature "Implementar a User Story 4567 do Azure DevOps" "LogSolutions-web"
/mogai-devops-pipeline:ship-feature "Portal interno para solicitação de veículos da frota, com aprovação do gestor e notificação por e-mail"
/mogai-devops-pipeline:clarify backlog.md
/mogai-devops-pipeline:atlas-audit Atlas.Net.Sdk.Security.Jwt
```
O guia avisa que esses exemplos são ilustrativos.

### B11. Limites e problemas
Não faz: merge, deploy/apply real, work item sem aprovação, decidir regra ambígua, usar tag `latest`, gravar credenciais. Tabela de solução de problemas (§13) e FAQ (§14).

---

## Parte C. Caso real no Claude Code (fonte: capturas)

| Captura | O que mostra |
|---|---|
| 01 | Autocomplete de `/mogai-devops-pipeline:ship-feature` e `evolve-feature` com as descrições do plugin |
| 02 | `/mogai-devops-pipeline:ship-feature "Criar módulo de consulta de Nomeações"` |
| 03 | **Gate de clarificação** de Nomeações: `backlog.md` com 7 histórias e 26 requisitos EARS, 17 decisões pendentes e 4 trechos vagos, perguntas fechadas em blocos (contexto, dados, LGPD, não funcionais, constituição/Atlas), ★ = recomendação; aponta falta de `CLAUDE.md` e `ATLAS.md` |
| 04 | Novo pedido: `ship-feature "Criar o CRUD de Menu ... no projeto PMS.Inbound.Manager, localizado em: C:\Repositories\Etaure\LogSolutions-api ..."` |
| 05 | Antes de rodar, o agente descobre que a API de Menu já existe (`MenusEndpoint.cs`, GET/POST/PUT/DELETE em `/api/v1/menus`, `MenuAdminService` de 375 linhas, commit 39e3c1bf) e pergunta o escopo |
| 06 | Recomenda trocar para `/evolve-feature` porque os repositórios já existem |
| 07 | Revisão das respostas: "Front + ajustes na API" e "evolve-feature" |
| 08 | `codebase-discovery` grava `docs\features\menus\discovery.md` (Angular 20, `@mogai/atlas-ui` ^1.0.38, sem CLAUDE.md/ATLAS.md, teste de Breadcrumb quebrado no web 75/76, API 820/820) e pergunta como seguir com a rede de segurança fraca |
| 09 | `arch.md`: sem mudança de infra, deploy API → web com rollback simples; backend e frontend em paralelo na branch `dev`, sem commit; backend com **839 de 839** testes (820 + 19 novos); **hook de segredo** confundiu `cancellationToken` e bloqueou, o parâmetro virou `ct` para revisão humana |
| 10 | "sim, lista os projetos do Azure DevOps": MCP lista 5 projetos (Port Solutions, SYMS, sandbox-agentes, Trytton, AtlasSDK) |
| 11 a 15 | Perguntas de criação dos cards: item pai (6292 Gestão de acesso), time (Port Solutions Ternium), iteração (sem iteração/backlog), granularidade (1 User Story por história) e revisão |
| 16 | 9 cards criados, **#6423 a #6431** (MN-01 a MN-09), estado New, EARS na descrição, Gherkin em Acceptance Criteria, tags; mapeamento MN-xx → #ID no `backlog.md` e `spec.md`; commits com `AB#6431`; próximo passo proposto: branches e PRs. Painel lateral mostra os artefatos: `docs/features/menus/{arch,backlog,deploy-runbook,discovery,spec}.md`, `docs/TECHNICAL.md`, `docs/USER-GUIDE.md` |
| 17 | Azure Boards: backlog "Port Solutions Ternium" com as histórias [Menus] MN-01 a MN-09 sob "Gestão de acesso" |

---

## Conceitos-chave (candidatos a slide)
1. Ecossistema Atlas (SDK + UI) — base comum de todos os sistemas — d3-fluxo (camadas)
2. Módulos do Atlas SDK — o que reusar antes de escrever — mapa de módulos
3. Atlas UI e componentes — padrão visual e produtividade — código `<at-table>` + GIFs
4. Arquitetura PMS — onde o Atlas entra numa solução real — d3-fluxo
5. Atlas Knowledge + Claude Code — por que o agente precisa de CLAUDE.md/ATLAS.md — d3-fluxo
6. Plugin em números (4/11/3/1/2) e princípios — metricas
7. Preparação do ambiente — ferramentas e comandos reais — codigo/checklist
8. Instalar, autenticar e verificar — `claude plugin ...`, `az login`, `/mcp` — codigo
9. ship-feature × evolve-feature — escolher o comando certo — comparacao
10. Fluxo da esteira de agentes — requisito até doc — d3-fluxo/timeline
11. Gate /clarify e EARS — nenhuma ambiguidade passa — captura 03 + tabela EARS
12. Agentes, modelos e permissões — quem faz o quê — tabela
13. Hooks fail-closed — guardrails — captura 09 (caso cancellationToken)
14. Azure DevOps via MCP — work items com aprovação — capturas 10 a 17
15. Artefatos e docs/ — rastreabilidade — árvore de pastas
16. Caso real CRUD de Menu — do pedido aos cards — sequência de capturas
17. Limites: o que não faz sozinho — humano no controle — lista
18. Troubleshooting — tabela §13

## Dados e números
- Plugin v1.5.0: 4 comandos, 11 agentes, 3 hooks, 1 MCP, 2 skills.
- Atlas backend: 23 pacotes NuGet; Atlas UI: 38 componentes, Angular 20, v1.0.37 (PPTX) / ^1.0.38 (discovery real).
- Caso Nomeações: 7 histórias, 26 requisitos EARS, 17 decisões pendentes, 4 trechos vagos.
- Caso Menu: discovery 5m 43s; arch 1m 57s; backend 13m 56s; testes API 820 → 839 (19 novos); web 75/76 de linha de base; 9 User Stories #6423 a #6431; 5 projetos no Azure DevOps.

## Narrativa sugerida
1. Problema: cada sistema reinventa infra e tela, e a IA sem contexto gera código fora do padrão.
2. Base: Atlas SDK + Atlas UI (o que já existe para reusar), com arquitetura PMS como exemplo.
3. Ponte: Atlas Knowledge + Claude Code.
4. Ferramenta: plugin `mogai-devops-pipeline` (números, princípios).
5. Mão na massa: preparar ambiente, instalar, autenticar, verificar.
6. Como funciona: comandos, esteira de agentes, /clarify, hooks, MCP, artefatos.
7. Prova: caso real CRUD de Menu nas capturas, do pedido aos 9 cards no Azure Boards.
8. Limites e próximo passo: humano no controle, o que fica fora (deploy, merge), troubleshooting e CTA.

## Lacunas
- **Divergência de módulos do Atlas:** o PPTX cita `Notification` e `Tenant`; o guia lista 23 pacotes sem `Notification`/`Tenant` e com `Data.Tenant.*`, `Cache`, `Plugin`, `License`. O guia diz que o escopo de cada pacote "foi inferido do nome". Preciso saber qual lista vale, ou mostro as duas com a fonte.
- **Versão do Atlas UI:** 1.0.37 no PPTX, ^1.0.38 no discovery real.
- **Sem capturas** da instalação (`claude plugin install`, `az login`, `/mcp`), das respostas ao `/clarify`, do test-runner/doc-writer final nem da criação de branches e PRs (a última captura só mostra o pedido "sim, cria as branches e abre os PRs"). Esses passos serão mostrados só com os comandos do guia.
- `docs/CONFIGURACAO-AMBIENTE-DEV.md` (Deploy Key) e a configuração do feed do Azure Artifacts não estão nas fontes.
- A esteira de 8 etapas do PPTX inclui deploy, homologação e observabilidade, que o plugin não executa; o deck precisa deixar essa fronteira clara.
- Os GIFs do PPTX somam cerca de 74 MB; para o deck, convém converter em vídeo (mp4/webm) preservando o conteúdo original.
