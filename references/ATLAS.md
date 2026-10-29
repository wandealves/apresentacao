# Atlas — Visão Geral, Vantagens e Desvantagens

> Explicação do ecossistema **Atlas**: o **Atlas .NET SDK** (`Atlassdk-Dotnet/`) para o back-end e a
> biblioteca Angular **Atlas UI** (`AtlasSDK-UI/`, pacote `@mogai/atlas-ui`) para o front-end.

---

## 1. O que é o Atlas

O Atlas é a **plataforma base interna** usada para construir os sistemas da empresa. Em vez de cada
projeto reinventar autenticação, acesso a dados, log, auditoria, multi-tenant e componentes de tela,
essas peças ficam centralizadas em duas bibliotecas versionadas e distribuídas por feeds privados:

| Camada | Projeto | Tecnologia | Distribuição |
|---|---|---|---|
| Back-end | **Atlas .NET SDK** (`Atlas.Net.Sdk.*`) | .NET 10, EF Core, ASP.NET Core | Feed NuGet privado (`atlas-net`) |
| Front-end | **Atlas UI** (`@mogai/atlas-ui`) | Angular 22 (standalone + signals), NG-ZORRO 22 | Azure Artifacts, feed npm `Mogai` |

```
┌───────────────────────────────┐        HTTP / JWT        ┌──────────────────────────────┐
│  App Angular                  │  ─────────────────────▶  │  API ASP.NET Core            │
│  └─ @mogai/atlas-ui (at-*)    │                          │  └─ Atlas.Net.Sdk.*          │
│     └─ NG-ZORRO + tema Falcon │                          │     ├─ Core (contratos)      │
└───────────────────────────────┘                          │     ├─ Data + Providers      │
                                                           │     ├─ Security.Jwt / Auth   │
                                                           │     ├─ Logger / Audit        │
                                                           │     └─ Queue / Cache / ...   │
                                                           └──────────────────────────────┘
```

---

## 2. Atlas .NET SDK

Conjunto de pacotes NuGet modulares — o projeto referencia só o que precisa. Todos são versionados
em **lockstep** (mesma versão, definida em `Directory.Build.props`) e têm como TFM `net10.0`.

### 2.1 Módulos

| Pacote | Responsabilidade |
|---|---|
| `Atlas.Net.Sdk.Core` | Raiz do grafo, **sem dependências**. Contratos e tipos comuns: `Entity`, *Result pattern* (`Result`, `Result<T>`, `PagedResult<T>`), `Specification<T>`, validação, `ITenantProvider`, `ICacheService<T>`, atributos de auditoria, extension methods. |
| `Atlas.Net.Sdk.Data` / `Data.Common` | Camada de dados sobre **Entity Framework Core** (repositórios, unit of work). |
| `Data.Providers.SQLServer` / `.PostgreSQL` / `.Oracle` | Implementações específicas de cada banco. |
| `Data.Tenant.SQL` / `Data.Tenant.Oracle` | Suporte a **multi-tenant** (resolução de schema/connection string por tenant). |
| `Atlas.Net.Sdk.Security.Jwt` | Autenticação/autorização com JWT, usuários, roles e permissões (ASP.NET Identity). |
| `Atlas.Net.Sdk.GoogleAuth` | Login com Google. |
| `Atlas.Net.Sdk.AspNet` | Helpers e integrações para ASP.NET Core (middlewares, DI). |
| `Atlas.Net.Sdk.Logger` (+ `.Mvc`, `.SQLServer`, `.PostgreSQL`, `.Oracle`) | Log centralizado com persistência em banco. |
| `Atlas.Net.Sdk.Audit` (+ `.Mvc`) | Auditoria de alterações em entidades marcadas com `[Auditable]`. |
| `Atlas.Net.Sdk.Cache` | Cache em memória atrás de `ICacheService<T>`. |
| `Atlas.Net.Sdk.Queue` | Processamento em fila / tarefas em segundo plano. |
| `Atlas.Net.Sdk.Message` | Envio de e-mail (MailKit / Azure Communication Email). |
| `Atlas.Net.Sdk.Plugin` | Carregamento dinâmico de plugins (inclusive scripts via Roslyn). |
| `Atlas.Net.Sdk.License` (+ `License.UI`) | Geração e validação de licenças (criptografia + HMAC, vínculo com MAC e data de expiração) e middleware de validação. O `License.UI` é um app desktop Avalonia, não publicado como pacote. |

A documentação detalhada de cada módulo está em `Atlassdk-Dotnet/docs/`, e há exemplos em
`Atlassdk-Dotnet/Samples/`.

---

## 3. Atlas UI (`@mogai/atlas-ui`)

Biblioteca de componentes Angular que padroniza a interface dos sistemas.

- **Componentes standalone** com **API de signals** (`input()` / `output()`), prefixo `at-`
  (ex.: `at-table`, `at-modal`, `at-date-picker`, `at-file-upload`, `at-sidebar`, `at-header`).
- Construída sobre o **NG-ZORRO** + tema visual próprio (**Falcon / Bootstrap 5.3**, FontAwesome,
  SimpleBar), distribuído junto com o pacote em `assets/`.
- Componentes de formulário implementam `ControlValueAccessor` → funcionam com `formControlName`,
  `[formControl]` e `[(ngModel)]`.
- ~45 componentes: formulário (input, checkbox, toggle, date/range picker, autocomplete, color/icon
  picker, upload), dados (table, table-custom, table-tree-data, dynamic-grid, pagination, transfer),
  layout (header, sidebar, tabs, accordion, collapse, cards, breadcrumb), feedback (toast, modal,
  modal-dialog, constraint-warning-modal) e visualização (audit-timeline, statistics-summary,
  chart-circular-progress).
- **Storybook** para catálogo e documentação visual dos componentes.
- **`atlas-mcp`**: servidor MCP (Python/FastMCP) para expor a biblioteca a agentes de IA.
- Publicação automática via **Azure Pipelines** ao fazer merge em `main` com bump de versão.

Uso básico:

```bash
npm install @mogai/atlas-ui
```

```ts
// app.config.ts
providers: [
  provideNzDateFnsAdapter({ locale: ptBR }),
  provideHttpClient(withXhr()),
  provideAnimationsAsync(),
]
```

Além disso, os CSS/JS do tema precisam ser adicionados ao `angular.json` (ver
`AtlasSDK-UI/ASSETS-USAGE.md`).

---

## 4. Vantagens

### Gerais (SDK + biblioteca)

- **Produtividade**: autenticação, log, auditoria, multi-tenant, acesso a dados e componentes de
  tela já estão prontos; o time foca na regra de negócio.
- **Padronização**: todos os sistemas seguem a mesma arquitetura, o mesmo vocabulário (`Result<T>`,
  `Entity`, `Specification`) e a mesma identidade visual. Fica mais fácil mover pessoas entre
  projetos e revisar código.
- **Correção uma vez, benefício em todos**: um bug ou falha de segurança corrigido no Atlas chega a
  todos os sistemas com uma atualização de versão.
- **Conhecimento centralizado**: decisões técnicas difíceis (multi-tenant, JWT, licenciamento,
  auditoria) ficam documentadas e testadas em um só lugar.

### Específicas do .NET SDK

- **Modularidade**: pacotes pequenos e independentes; o `Core` não tem dependências, então não
  arrasta nada desnecessário.
- **Multi-banco**: SQL Server, PostgreSQL e Oracle com a mesma API — trocar de banco exige menos
  reescrita.
- **Multi-tenant pronto**: resolução de tenant e schema já resolvida via `ITenantProvider`.
- **Versão única (lockstep)**: todos os pacotes na mesma versão evitam combinações incompatíveis.
- **Recursos transversais prontos**: auditoria automática por atributo, log em banco, e-mail, fila,
  cache, plugins e licenciamento.

### Específicas da biblioteca Angular

- **Consistência visual** entre todos os front-ends, sem copiar e colar componentes.
- **Angular moderno**: standalone + signals, importável por componente (melhor *tree-shaking*).
- **Integração nativa com Reactive Forms** via `ControlValueAccessor`.
- **Storybook** para ver e testar componentes isoladamente, e documentação pensada também para IA
  (`docs/mogai-atlas-ui.md` + `atlas-mcp`).
- **CI/CD automatizado**: versão nova é publicada pelo pipeline, sem passo manual.

---

## 5. Desvantagens e riscos

### Gerais

- **Acoplamento à plataforma**: os sistemas passam a depender do Atlas. Uma decisão ruim no SDK se
  espalha para todos os projetos, e sair dele depois é caro.
- **Gargalo de manutenção**: se o time do Atlas for pequeno, novas necessidades ficam esperando
  uma versão da biblioteca. Há tentação de "contornar" o SDK em cada projeto, o que quebra a
  padronização.
- **Breaking changes em cascata**: uma versão major (ex.: migração para Angular 22 ou .NET 10)
  obriga todos os consumidores a migrar juntos.
- **Curva de aprendizado**: além de .NET/Angular, o desenvolvedor precisa aprender as convenções e
  abstrações próprias do Atlas.
- **Código proprietário e feeds privados**: sem conhecimento ou documentação da comunidade (Stack
  Overflow etc.); instalação exige configurar e autenticar no NuGet `atlas-net` e no Azure
  Artifacts.
- **Documentação desatualizada**: existem divergências hoje entre docs e código, por exemplo:
  - o `README` do .NET SDK pede **.NET 8**, mas os projetos estão em **`net10.0`**;
  - `docs/mogai-atlas-ui.md` fala em **Angular 20 / v1.0.37**, mas o `package.json` está em
    **v2.0.0 / Angular 22**;
  - o `README` do SDK ainda tem o aviso "Documentação em Construção".

### Específicas do .NET SDK

- **Abstração sobre EF Core**: repositórios genéricos podem esconder recursos do EF ou dificultar
  consultas otimizadas; às vezes é preciso "furar" a abstração.
- **Lockstep**: atualizar um único pacote implica atualizar todos.
- **Cobertura de testes parcial**: há projetos de teste só para `Core`, `Audit`, `Plugin` e
  `Security.Jwt`; módulos como Data, Providers, Tenant, Logger e Queue não têm testes dedicados.
- **Muitas dependências de terceiros** somadas (EF Core, Identity, Roslyn, MailKit, Azure, Google,
  Avalonia), aumentando superfície de atualização e de segurança.

### Específicas da biblioteca Angular

- **Duas camadas de dependência**: depende do NG-ZORRO; uma major do NG-ZORRO ou do Angular força
  uma major do Atlas UI (peer `^22.0.0` exige Angular 22 no consumidor).
- **Tema pesado e configuração manual**: Bootstrap, Popper, FontAwesome, Lodash, List.js etc. são
  carregados globalmente via `angular.json`, em ordem específica. Isso aumenta o bundle e mistura
  dois sistemas de estilo (Ant Design + Bootstrap), o que pode gerar conflitos de CSS.
- **Sem testes unitários** nos componentes (nenhum `*.spec.ts` em `src/lib`); a validação depende
  do Storybook e de testes manuais.
- **Customização limitada**: se um componente `at-*` não expõe a opção necessária, o projeto precisa
  esperar uma nova versão ou usar o NG-ZORRO diretamente, perdendo a padronização.

---

## 6. Quando faz sentido usar

| Use o Atlas quando… | Considere não usar quando… |
|---|---|
| O sistema é interno/corporativo e segue o padrão da empresa. | O projeto é um protótipo isolado ou tem stack diferente. |
| Precisa de multi-tenant, auditoria, JWT, log e licença. | Precisa de controle fino de performance de acesso a dados. |
| A interface deve seguir a identidade visual comum. | A interface tem design próprio, muito diferente do tema Falcon. |
| O time consegue acompanhar as versões do Atlas. | Não há como acompanhar upgrades de Angular/.NET no ritmo do Atlas. |

## 7. Recomendações

1. **Atualizar a documentação** para refletir .NET 10 e Angular 22 / v2.0.0.
2. **Ampliar testes**: módulos de dados/tenant no SDK e `*.spec.ts` nos componentes do Atlas UI.
3. **Manter um CHANGELOG** com breaking changes e guias de migração (como
   `docs/migracao-angular-22.md`) a cada versão major.
4. **Definir um processo de contribuição** para que os times de produto proponham melhorias no
   Atlas em vez de criar contornos locais.
