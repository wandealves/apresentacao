# Plano refinado: Atlas SDK e plugin Mogai no fluxo DevOps

Template `mira-default` (título em cima, animação ocupando o quadro 16:9), tema `corporate-blue`.
Regras de conteúdo: comandos copiados literalmente das fontes; capturas e GIFs originais, sem edição de conteúdo; nada além do que está em `mira/briefing.md`.

> **Atualização 2026-10-26:** removidos a pedido do autor os slides 10 (Seis ferramentas antes do primeiro comando), 14 (17 decisões que ninguém deveria adivinhar), 15 (Na dúvida, o hook bloqueia) e 16 (O agente que descobriu a API pronta). O deck tem 16 slides; a numeração abaixo é a do plano original.

## Arco
- Abertura (1-2): os números e a promessa da plataforma.
- Base (3-6): o que o Atlas já entrega.
- CTA (7): auditar antes de padronizar.
- Ponte (8-9): o agente precisa de contexto; o plugin entrega a esteira.
- Mão na massa (10-12): ambiente, instalação, escolha do comando.
- Como funciona (13-15): esteira, /clarify, guardrails.
- Prova (16-17): caso real CRUD de Menu.
- Clímax (18-19): onde o humano decide; resumo.
- Fechamento (20): o próximo pedido.

---

## Slide 1 (grid de números)
- **Título:** Do pedido ao backlog, com o Atlas
- **Subtítulo:** SDK, UI e uma esteira de agentes no Claude Code
- **Conteúdo:** 4 números que entram contando: **23** pacotes NuGet (Atlas backend) · **38** componentes (Atlas UI) · **4** comandos · **11** agentes (plugin `mogai-devops-pipeline` v1.5.0)
- **Ícone:** layers
- **Animação:** contadores sobem; depois os 4 blocos respiram e uma linha de fluxo percorre os blocos em loop.

## Slide 2 (D3 fluxo)
- **Título:** Duas bibliotecas, uma aplicação final
- **Subtítulo:** Atlas SDK no backend .NET, Atlas UI no Angular
- **Conteúdo:** nós "Atlas SDK · .NET 10 | C#" e "Atlas UI · Angular" convergindo em "Aplicação Final". Pilares do PPTX: Segurança (JWT, RBAC, multi-tenant, auditoria), Integração (mensageria, logging), Mais produtividade (padrões, reutilização, consistência).
- **Ícone:** boxes
- **Animação:** partículas saem dos dois nós e alimentam a aplicação final, em loop.

## Slide 3 (grid)
- **Título:** Antes de escrever, veja se o Atlas já faz
- **Subtítulo:** Módulos `Atlas.Net.Sdk.*` no feed privado do Azure Artifacts
- **Conteúdo (PPTX s2):** Core (núcleo e configurações base) · Audit (auditoria e rastreabilidade) · Data (EF Core, Repositories/Query, Specification Query) · Provedores (Oracle, PostgreSQL, SQL Server, Data.Common) · Especializados (Security.Jwt, GoogleAuth, Message, Queue/RabbitMQ, Logger, AspNet, Notification, Tenant).
- **Rodapé:** "Guia do plugin: monorepo `Atlassdk-Dotnet`, 23 pacotes. O que cada um cobre foi inferido do nome; confirme na documentação."
- **Ícone:** package
- **Animação:** os módulos encaixam como peças numa base; um pulso percorre as peças em loop.

## Slide 4 (código)
- **Título:** Uma tabela completa em cinco linhas
- **Subtítulo:** `@mogai/atlas-ui` · Angular 20 standalone · NG-ZORRO
- **Conteúdo:** trecho real do PPTX digitado:
```html
<at-table
  [data]="usuarios"
  [columns]="colunas"
  [loading]="carregando"
  (onPageChange)="carregarDados($event)">
</at-table>
```
  Categorias ao lado: Formulários · Tabelas e Dados · Layout e Navegação · Visualização de Dados · Interação e Feedback · Especializados.
- **Ícone:** code
- **Animação:** digitação do código; depois o cursor pisca e a tabela renderizada ao lado pagina em loop (linhas Ana Silva, Bruno Costa, Carla Souza do PPTX).

## Slide 5 (imagem/vídeo)
- **Título:** Componentes reais, telas reais
- **Subtítulo:** Storybook e Port Solutions construídos com o Atlas UI
- **Conteúdo:** os 4 GIFs do PPTX convertidos em vídeo, em rodízio com legenda: Autocomplete · Atendimentos · CapacityProgressCard · Editor de despacho ferroviário.
- **Assets:** `assets/video/atlas-ui-autocomplete.mp4`, `atlas-ui-atendimentos.mp4`, `atlas-ui-capacity-card.mp4`, `atlas-ui-editor-ferroviario.mp4` (convertidos de `references/Apresentação SDK.pptx`, image4 a image7).
- **Ícone:** monitor-play
- **Animação:** moldura entra; vídeos em loop com troca a cada clipe.

## Slide 6 (D3 fluxo)
- **Título:** Onde o Atlas entra numa arquitetura real
- **Subtítulo:** PMS: do gateway às filas, com o SDK por baixo
- **Conteúdo:** Usuário → Gateway (autenticação, autorização, rate limiting) → External Integration API · PMS API · IAM API ↔ Integration · PMS · IAM Service → Queue Service + MinIO ↔ RabbitMQ (Fila 1..N). Faixa Atlas SDK: Core, Data, Security.Jwt, Queue, Audit, Tenant. Dados: Outbound IAM, Outbound PMS, Oracle.
- **Ícone:** network
- **Animação:** requisições viajam pelo fluxo; mensagens assíncronas enchem e esvaziam as filas em loop.

## Slide 7 (CTA)
- **Título:** Audite antes de padronizar
- **Subtítulo:** O Atlas veio de uma aquisição e está sob avaliação
- **Conteúdo:** `/mogai-devops-pipeline:atlas-audit Atlas.Net.Sdk.Security.Jwt` → laudo `atlas-audit-<modulo>.md` com veredito **PRONTO** · **PRONTO COM RESSALVAS** · **NÃO RECOMENDADO**. Fluxo só de leitura.
- **Ícone:** shield-check
- **Animação:** comando digitado; um scanner passa sobre o módulo e o selo do veredito alterna entre os três em loop.

## Slide 8 (D3 fluxo)
- **Título:** O agente só acerta o padrão que conhece
- **Subtítulo:** Atlas Knowledge dá ao Claude Code o contexto da empresa
- **Conteúdo:** Desenvolvedor (define o objetivo e revisa) ↔ Claude Code (entende o contexto, consulta a documentação Atlas, aplica os padrões, gera e refatora código) ↔ Atlas Knowledge: `CLAUDE.md` e `AGENTS.md` · Documentação Atlas (spec.md por pacote, patterns, recipes, ADRs) · Padrões de Projeto.
- **Ícone:** book-open
- **Animação:** documentos alimentam o agente, que devolve código ao desenvolvedor, num ciclo contínuo.

## Slide 9 (lista)
- **Título:** Uma esteira inteira dentro do Claude Code
- **Subtítulo:** `mogai-devops-pipeline` v1.5.0
- **Conteúdo:** **4** comandos · **11** agentes · **3** hooks · **1** servidor MCP (Azure DevOps) · **2** skills (SDD e DDD). Princípios: a spec é o artefato central · reusar o Atlas primeiro · nenhuma ambiguidade passa adiante · humano no controle · segurança fail-closed.
- **Ícone:** workflow
- **Animação:** itens entram em cascata; números pulsam em sequência em loop.

## Slide 10 (tabela)
- **Título:** Seis ferramentas antes do primeiro comando
- **Subtítulo:** O plugin roda no Claude Code CLI, não no claude.ai web
- **Conteúdo:** Claude Code (CLI) · Node.js `winget install OpenJS.NodeJS.LTS` · Azure CLI `winget install Microsoft.AzureCLI` · jq `winget install jqlang.jq` · git · .NET SDK / npm do projeto.
- **Alerta:** "Sem `jq` e sem `node`, os hooks bloqueiam toda escrita de arquivo. É de propósito."
- **Ícone:** wrench
- **Animação:** cada linha recebe um check em sequência; o alerta pisca suave em loop.

## Slide 11 (código)
- **Título:** Instale, autentique, confira
- **Subtítulo:** Depois de configurar a Deploy Key de leitura
- **Conteúdo:**
```bash
claude plugin marketplace add git@github.com-mogai-devops-pipeline:mogaigit/mogai-devops-pipeline.git
claude plugin install mogai-devops-pipeline@mogai-devops-pipeline
az login
```
```
/mcp              # deve listar o azure-devops como conectado
/reload-plugins   # recarrega depois de atualizar o plugin
```
- **Ícone:** terminal
- **Animação:** digitação por etapa; o indicador "azure-devops conectado" pulsa em loop. Sem saída simulada dos comandos.

## Slide 12 (comparativo)
- **Título:** Projeto novo ou sistema em produção?
- **Subtítulo:** O comando certo muda o caminho inteiro
- **Conteúdo:** `ship-feature` (greenfield, do zero) × `evolve-feature` (existente: `codebase-discovery` primeiro, modo delta, contratos preservados, proteção contra regressão). Captura 01 com o autocomplete real.
- **Asset:** `assets/img/plugin-mogai-01.png`
- **Ícone:** git-branch
- **Animação:** os dois caminhos se acendem alternadamente; a captura entra por baixo.

## Slide 13 (timeline)
- **Título:** Do requisito à documentação, etapa por etapa
- **Subtítulo:** Cada agente entrega um artefato que você revisa
- **Conteúdo:** requirements-analyst → `backlog.md` · /clarify (GATE) · spec-writer → `spec.md` · devops-architect → `arch.md` · backend-implementer ‖ frontend-implementer → código · devops-engineer → IaC + CI/CD (para antes de apply/deploy) · test-runner → testes · doc-writer → `docs/TECHNICAL.md`, `docs/USER-GUIDE.md` · resumo para revisão humana.
- **Ícone:** git-commit-horizontal
- **Animação:** um token percorre a esteira, para no gate e se divide em dois no paralelo, em loop.

## Slide 14 (imagem)
- **Título:** 17 decisões que ninguém deveria adivinhar
- **Subtítulo:** O /clarify para a esteira e pergunta, com opções fechadas
- **Conteúdo:** captura 03 real (Nomeações: 7 histórias, 26 requisitos EARS, 17 decisões pendentes, 4 trechos vagos). Ao lado, EARS: Ubíquo "O sistema deve…" · Evento "Quando…" · Estado "Enquanto…" · Indesejado "Se…, então…" · Opcional "Onde…".
- **Asset:** `assets/img/plugin-mogai-03.png`
- **Ícone:** circle-help
- **Animação:** zoom lento passeando pelos blocos A a E da captura em loop.

## Slide 15 (D3)
- **Título:** Na dúvida, o hook bloqueia
- **Subtítulo:** Três guardrails automáticos, dois deles fail-closed
- **Conteúdo:** `block-secret.sh` (antes de Write/Edit, bloqueia) · `format-on-save.sh` (depois, prettier/black, pula se falhar) · `require-tests-pass.sh` (fim do test-runner, bloqueia). Caso real: o hook confundiu `cancellationToken` com segredo; o parâmetro virou `ct` e a troca ficou para revisão humana.
- **Ícone:** shield-alert
- **Animação:** escritas passam pelos três portões; uma é barrada no primeiro, em loop.

## Slide 16 (timeline)
- **Título:** O agente que descobriu a API pronta
- **Subtítulo:** Caso real: CRUD de Menu no LogSolutions
- **Conteúdo:** 04 pedido com `ship-feature` · 05 achado: a API de Menu já existe em `/api/v1/menus` · 06/07 recomenda `evolve-feature`, escopo "Front + ajustes na API" · 08 discovery: web 75/76, API 820/820 · 09 backend e frontend em paralelo na branch `dev`, sem commit; API com 839 de 839 testes (820 + 19 novos).
- **Assets:** `assets/img/plugin-mogai-04.png` a `plugin-mogai-09.png`
- **Ícone:** search-check
- **Animação:** a timeline avança e cada marco abre a captura correspondente, em loop.

## Slide 17 (imagem)
- **Título:** Nove cards criados, nenhum sem aprovação
- **Subtítulo:** Do terminal ao Azure Boards pelo MCP
- **Conteúdo:** 10 listar projetos da organização Mogai · 11-15 escolher pai (6292 Gestão de acesso), time (Port Solutions Ternium), iteração (backlog) e granularidade (1 User Story por história) · 16 cards #6423 a #6431, commits com `AB#` · 17 backlog no Azure Boards.
- **Assets:** `assets/img/plugin-mogai-10.png` a `plugin-mogai-17.png`
- **Ícone:** kanban
- **Animação:** sequência de capturas em rodízio; o contador de cards sobe até 9.

## Slide 18 (comparativo)
- **Título:** Onde a esteira para e você decide
- **Subtítulo:** O plugin prepara; merge e deploy continuam com você
- **Conteúdo:** esteira de 8 etapas do PPTX (Planejamento, Análise e Implementação, Controle de Código, Revisão Humana, CI/CD e Segurança, Homologação, Produção, Observabilidade) com a fronteira na Revisão Humana. Não faz sozinha: merge de PR · deploy ou apply real · work item sem aprovação · decidir regra ambígua · tag `latest` · gravar credenciais.
- **Ícone:** hand
- **Animação:** a esteira corre e para na barreira humana; um carimbo de aprovação libera a passagem em loop.

## Slide 19 (lista)
- **Título:** Seis movimentos do pedido à entrega revisada
- **Conteúdo:** 1 Reusar o Atlas e auditar antes de padronizar · 2 Preparar o ambiente (Node, Azure CLI, jq) · 3 Escolher `ship-feature` ou `evolve-feature` · 4 Responder ao `/clarify` · 5 Deixar os hooks guardarem segredos e testes · 6 Aprovar cada escrita no Azure DevOps.
- **Ícone:** list-checks
- **Animação:** itens marcados em sequência, com um destaque que percorre a lista em loop.

## Slide 20 (CTA)
- **Título:** Seu próximo pedido já cabe na esteira
- **Subtítulo:** Comece por uma evolução pequena num sistema existente
- **Conteúdo:**
```
/mogai-devops-pipeline:evolve-feature "Adicionar filtro por período (data inicial/final) na listagem de changelogs" "LogSolutions-web"
```
  Documentação do plugin: `~/.claude/plugins/cache/mogai-devops-pipeline/mogai-devops-pipeline/1.5.0/`
- **Ícone:** rocket
- **Animação:** comando digitado; o cursor pisca e um brilho percorre o botão em loop.

---

## Atualização 2026-10-27: introdução ao Atlas (7 slides novos no início)

Inseridos antes do slide original "Do pedido ao backlog, com o Atlas", sem alterar nenhum slide existente. Fonte factual: `references/ATLAS.md` (cópia do `ATLAS.md` da raiz); o slide dos agentes usa também a tabela §4 do `references/GUIA-MOGAI-DEVOPS-PIPELINE.md`. HTML, CSS e JS ficam em blocos próprios marcados "INTRODUÇÃO AO ATLAS" (ids `a00`..`a06`). Versão anterior: `mira/backup/index.antes-intro-atlas.html`.

| # | Título | Animação (loop) |
|---|---|---|
| 0 | Capa: Atlas + Claude Code (logo `assets/img/logo.png`) | a fundação de módulos se assenta, os sistemas sobem; uma "nova versão" percorre a base e cada sistema se atualiza quando ela passa por baixo |
| 1 | O que é o Atlas | requisição sai do Atlas UI, viaja por HTTP/JWT, passa por Security.Jwt, Data e Logger/Audit e volta como `Result<T>` |
| 2 | Por que o Atlas existe | 3 projetos constroem as mesmas 6 peças, cada um do seu jeito (18 construções); o Atlas constrói 6 uma vez e atende os 3 |
| 3 | O que a empresa ganha | bug no Atlas aparece em todos os sistemas; corrigido uma vez, a versão sobe e a correção desce em onda |
| 4 | O que o time ganha | barra de esforço: as 5 peças de infraestrutura se comprimem em "Atlas" e a regra de negócio se expande (proporções ilustrativas) |
| 5 | Evoluir e adotar o Atlas | ciclo de contribuição: time propõe, Atlas publica, todos atualizam; "contorno local" fica fora do ciclo |
| 6 | Agentes: especialistas com limites claros | 11 agentes em 3 faixas de acesso; em loop, 4 relações com o Atlas: descobrir, compor, auditar, contribuir |

### Ajustes 2026-10-27 (pedidos do autor)
- **Slide 1 "O que é o Atlas"**: refeito para leigos com a comparação do shopping. A estrutura comum (segurança, câmeras, depósito, fachada) vem pronta e cada loja (sistema) cuida do que vende (regra de negócio), com uma legenda "No shopping → No Atlas". Versão técnica anterior em `mira/backup/index.intro-atlas-v1.html`.
- **Slide 6 "Claude Code e agentes de IA"** (antes "Agentes: especialistas com limites claros"): só a mensagem de que vamos usar o Claude Code e agentes de IA. Você pede, o Claude Code reparte o trabalho, 4 agentes reutilizam o Atlas e entregam partes, e você revisa e aprova. A lista dos 11 agentes saiu (já está nos slides originais). Versão anterior em `mira/backup/index.intro-atlas-v2.html`.
- **Slide 1 "O que é o Atlas", v3**: o autor pediu só a explicação, sem comparação. Agora tem uma definição em uma frase e as duas partes em linguagem simples: Atlas .NET SDK ("a parte que roda no servidor") e Atlas UI ("as telas que o usuário vê"), cada uma com 5 peças prontas. Versão do shopping em `mira/backup/index.intro-atlas-v3.html`.
- **Slide novo "Harness + Claude Code"** (depois de "Onde o Atlas entra numa arquitetura real", antes de "Audite antes de padronizar"): o modelo (Claude) no centro e o harness (Claude Code) em volta, com quatro estações (contexto, ferramentas, permissões e hooks, extensões). Em loop, a tarefa faz o ciclo ler contexto → decidir → permissão → ferramenta → resultado; a cada três voltas o hook block-secret.sh barra a ação. As fontes não definem "harness": o conceito é explicado com o que é público sobre o Claude Code, e os exemplos (CLAUDE.md, block-secret.sh, MCP do Azure DevOps) vêm do guia do plugin. Animação h01. Versão anterior: `mira/backup/index.intro-atlas-v4.html`.
- **Removido a pedido do autor** o slide original "Audite antes de padronizar" (plano original slide 7, svg `s07`): a seção e a função `s07` saíram e o registro em `ligar()` foi atualizado. O deck tem 23 slides. Versão anterior: `mira/backup/index.antes-remover-audite.html`.
- **Reordenado a pedido do autor**: "Evoluir e adotar o Atlas" foi para depois de "Onde o Atlas entra numa arquitetura real", e "Seu próximo pedido já cabe na esteira" foi para depois de "Do requisito à documentação, etapa por etapa". Conteúdo das seções inalterado. Versão anterior: `mira/backup/index.antes-reordenar.html`.
- **Slide novo "Conclusão: Atlas + Claude Code"** (último, depois de "Seis movimentos do pedido à entrega revisada"): as quatro ideias em cadeia (Atlas, Claude Code, agentes e esteira, você no controle), o próximo passo com a sintaxe do `evolve-feature` do guia e o logo. Em loop, um pedido percorre a cadeia, recebe o selo de aprovado em "você" e o próximo passo acende. Animação c01. Versão anterior: `mira/backup/index.antes-conclusao.html`.
