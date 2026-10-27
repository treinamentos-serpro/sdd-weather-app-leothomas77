# Backlog de Tarefas — Weather App

Tarefas derivadas de `plans/weather-app-plan.md` e rastreadas a `specs/weather-app-spec.md`. T-01 a T-03 são bloqueadoras: resolva e registre essas decisões na spec antes de implementar. A sequência segue tipos → funções puras → services → hook → componentes → integração → testes → hardening.

Tipos: **UI** para interface/orquestração, **Data** para contratos/regras/integração, **Test** para testes, **Infra** para metas e qualidade compartilhadas.

Prioridade: **P0** para bloqueadores, primeira fatia utilizável e gates de release; **P1** para completar o MVP depois da primeira fatia; **P2** para melhorias opcionais/deferíveis. Não há tarefas P2 neste backlog porque todas atendem requisitos ou gates do escopo atual.

Tamanho relativo: **P** pequeno e focado, **M** médio, **G** grande ou com maior risco/integração. Se uma tarefa G ultrapassar uma unidade testável, divida-a antes de iniciar.

## Entrega 1 — Decisões de produto e contratos

### T-01 — Aprovar escopo e conteúdo meteorológico

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-01, FR-02, FR-03; AC-01.2, AC-02.1, AC-03.1
- **Descrição:** Fechar campos de geocoding, geolocalização opcional, variáveis de clima atual e granularidade/variáveis da previsão.
- **Critérios de aceite:**
  - A spec enumera os campos usados para distinguir cidades homônimas.
  - A spec registra se haverá geolocalização automática ou somente busca manual.
  - A spec lista campos mínimos do clima atual e da previsão e define previsão diária, horária ou ambas.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-02 — Aprovar políticas de dados e unidades

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-02, FR-03, FR-04, NFR-04, NFR-05; AC-02.5, AC-03.5, AC-04.1–AC-04.4
- **Descrição:** Definir frescor dos dados, fuso e formatos de data/hora, timeout/retry, persistência local da unidade e arredondamento.
- **Critérios de aceite:**
  - A spec define frequência de atualização e idade máxima para dados serem chamados de atuais.
  - A spec fixa timeout e política de nova tentativa para geocoding e forecast.
  - A spec define timezone/formatos usados para o período “hoje + quatro dias”.
  - A spec decide persistência local da unidade e precisão/arredondamento de temperaturas.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-03 — Aprovar metas não funcionais

- **Tipo:** Infra
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** NFR-01, NFR-02, NFR-03, NFR-06; AC-05.1–AC-05.3
- **Descrição:** Definir matrizes suportadas e metas mensuráveis de responsividade, acessibilidade, compatibilidade e desempenho.
- **Critérios de aceite:**
  - A spec lista viewports/dispositivos móveis e navegadores/sistemas suportados.
  - A spec define padrão e nível de conformidade de acessibilidade.
  - A spec define metas de latência, percentis e condições de medição.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-04 — Definir contratos TypeScript

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-01–FR-04; AC-01.1–AC-01.7, AC-02.1–AC-02.5, AC-03.1–AC-03.5, AC-04.1–AC-04.4
- **Descrição:** Criar `Unit`, `City`, `CurrentWeather`, `ForecastDay`, `WeatherData` e tipos de status/erro, conforme decisões aprovadas.
- **Critérios de aceite:**
  - Os tipos compilam em TypeScript strict e correspondem aos campos aprovados na spec.
  - Campos que podem faltar em resposta parcial representam indisponibilidade explicitamente.
  - Temperaturas têm Celsius como unidade canônica e `Unit` é `'celsius' | 'fahrenheit'`.
- **Dependências:** T-01, T-02, T-03.
- **Arquivos prováveis:** `src/types/weather.ts`.

## Entrega 2 — Funções puras

### T-05 — Implementar funções puras de clima

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-02, FR-03, FR-04, NFR-05; AC-02.3, AC-03.4, AC-04.1–AC-04.4
- **Descrição:** Implementar derivação de temperatura, mapeamento de códigos WMO e formatação a partir dos contratos aprovados.
- **Critérios de aceite:**
  - Temperaturas Celsius são formatadas como Celsius ou derivadas para Fahrenheit sem efeitos colaterais.
  - `null` permanece indisponível e nunca gera valor numérico.
  - Códigos WMO conhecidos e desconhecidos produzem o resultado definido na spec.
  - Formatos e arredondamento seguem T-02.
- **Dependências:** T-04.
- **Arquivos prováveis:** `src/lib/weather.ts`.

## Entrega 3 — Services

### T-06 — Implementar service de geocoding

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-01, NFR-04; AC-01.1–AC-01.7
- **Descrição:** Consultar Open-Meteo Geocoding, mapear resultados para `City` e distinguir vazio de falha.
- **Critérios de aceite:**
  - A URL codifica o termo e usa os parâmetros de geocoding aprovados em T-01.
  - Cada resultado mapeia para `City`, incluindo os campos de distinção aprovados.
  - Lista vazia retorna `[]`; HTTP não-2xx, rede, timeout e JSON inválido retornam erros tipados distintos.
- **Dependências:** T-01, T-04, T-05.
- **Arquivos prováveis:** `src/services/openMeteo.ts`.

### T-07 — Implementar service de forecast

- **Tipo:** Data
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-02, FR-03, FR-04, NFR-04; AC-02.1–AC-02.5, AC-03.1–AC-03.5
- **Descrição:** Consultar Open-Meteo Forecast com dados atuais/diários aprovados e mapear para `WeatherData`.
- **Critérios de aceite:**
  - A URL usa coordenadas da `City` e os parâmetros `current`, `daily`, timezone, unidades e período aprovados.
  - Campos atuais mapeiam para `CurrentWeather`; cada índice comum dos arrays diários mapeia para um `ForecastDay` associado à data correta.
  - Campos opcionais ausentes viram indisponíveis; resposta estruturalmente inválida retorna erro tipado.
- **Dependências:** T-01, T-02, T-04, T-05.
- **Arquivos prováveis:** `src/services/openMeteo.ts`.

## Entrega 4 — Hook e estado

### T-08 — Implementar hook `useWeather`

- **Tipo:** UI
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-01–FR-04, NFR-04; AC-01.6–AC-01.7, AC-02.2, AC-03.2, AC-04.4
- **Descrição:** Orquestrar busca, seleção, consulta, unidade e estados independentes sem estado global.
- **Critérios de aceite:**
  - Busca e clima mantêm status independentes `idle/loading/success/error/empty`.
  - Selecionar uma cidade chama forecast; geocoding vazio não chama forecast.
  - Alterar `Unit` recalcula apresentação sem chamar services.
  - Resposta atrasada de uma cidade anterior não sobrescreve a seleção atual.
- **Dependências:** T-06, T-07.
- **Arquivos prováveis:** `src/hooks/useWeather.ts`.

## Entrega 5 — Componentes

### T-09 — Criar busca e resultados de cidades

- **Tipo:** UI
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-01, NFR-02; AC-01.1–AC-01.6
- **Descrição:** Criar componentes de entrada de busca e seleção de resultados usando props e callbacks.
- **Critérios de aceite:**
  - Termo válido pode ser submetido e cidade selecionada dispara callback com o `City` correspondente.
  - Termo vazio ou composto apenas por espaços não dispara busca.
  - Resultados exibem o contexto geográfico aprovado em T-01 e são operáveis por teclado com nomes acessíveis.
- **Dependências:** T-04, T-08.
- **Arquivos prováveis:** `src/components/CitySearch.tsx`, `src/components/CityResults.tsx`.

### T-10 — Criar cartão de clima atual

- **Tipo:** UI
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-02, FR-04, NFR-01, NFR-02; AC-02.1–AC-02.4
- **Descrição:** Exibir dados atuais, cidade, unidade e campos parcialmente indisponíveis.
- **Critérios de aceite:**
  - Os campos definidos em T-01 são exibidos para a cidade recebida por props e temperaturas respeitam a unidade ativa.
  - `null` aparece como indisponível, sem valor substituto apresentado como dado real.
  - O cartão atende aos viewports definidos em T-03 sem ocultar controles ou dados essenciais.
- **Dependências:** T-04, T-05, T-08.
- **Arquivos prováveis:** `src/components/CurrentWeatherCard.tsx`.

### T-11 — Criar lista de previsão

- **Tipo:** UI
- **Prioridade:** P1
- **Tamanho:** M
- **Requisitos:** FR-03, FR-04, NFR-01, NFR-02; AC-03.1–AC-03.4
- **Descrição:** Exibir datas e variáveis de previsão aprovadas, incluindo dados parciais.
- **Critérios de aceite:**
  - Cada dia recebido é exibido uma vez, em ordem de data local, com variáveis aprovadas em T-01.
  - Temperaturas usam a unidade ativa; campos ou dias indisponíveis são identificados sem valores inventados.
  - A lista atende aos viewports definidos em T-03 sem ocultar informação essencial.
- **Dependências:** T-04, T-05, T-08.
- **Arquivos prováveis:** `src/components/ForecastList.tsx`.

### T-12 — Criar alternância de unidade

- **Tipo:** UI
- **Prioridade:** P1
- **Tamanho:** P
- **Requisitos:** FR-04, NFR-02; AC-04.1–AC-04.4
- **Descrição:** Criar controle acessível que atualiza a unidade por callback.
- **Critérios de aceite:**
  - Controle expõe a unidade atual e chama o callback com `'celsius'` ou `'fahrenheit'` ao selecionar a outra opção.
  - Pode ser operado por teclado e localizado por role e nome acessível.
  - Alternar unidade não inicia requisição de rede.
- **Dependências:** T-04, T-08.
- **Arquivos prováveis:** `src/components/UnitToggle.tsx`.

### T-13 — Criar estados visuais da aplicação

- **Tipo:** UI
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-01–FR-03, NFR-02, NFR-04; AC-01.3, AC-01.7, AC-02.4–AC-02.5, AC-03.3, AC-03.5
- **Descrição:** Apresentar loading, erro, vazio e dados parciais para busca e clima.
- **Critérios de aceite:**
  - Geocoding sem resultados e erro de rede/API apresentam mensagens distintas.
  - Erro de current ou forecast identifica a operação e ação de retry chama o callback correspondente.
  - Loading é anunciado como status acessível; empty, error e indisponibilidade parcial têm feedback identificável.
- **Dependências:** T-04, T-08.
- **Arquivos prováveis:** `src/components/WeatherStatus.tsx`.

## Entrega 6 — Integração vertical por etapas

### T-14 — Integrar primeira fatia vertical: busca e clima atual

- **Tipo:** UI
- **Prioridade:** P0
- **Tamanho:** M
- **Requisitos:** FR-01, FR-02, NFR-01, NFR-02, NFR-04
- **Descrição:** Compor busca, seleção, clima atual e estados da operação em `App` para entregar o primeiro fluxo utilizável.
- **Critérios de aceite:**
  - O fluxo busca → seleção → clima atual apresenta dados da cidade selecionada.
  - Loading, vazio, erro e retry de busca e clima atual são distintos e acessíveis.
  - Busca e clima atual permanecem utilizáveis nos viewports aprovados em T-03.
- **Dependências:** T-08, T-09, T-10, T-13.
- **Arquivos prováveis:** `src/App.tsx`, `src/components/`.

### T-21 — Estender integração com previsão e alternância de unidade

- **Tipo:** UI
- **Prioridade:** P1
- **Tamanho:** M
- **Requisitos:** FR-03, FR-04, NFR-01, NFR-02, NFR-05, NFR-06
- **Descrição:** Completar a primeira fatia com a previsão de cinco dias e o controle de unidade.
- **Critérios de aceite:**
  - A cidade selecionada apresenta hoje e os quatro dias seguintes com as variáveis aprovadas em T-01.
  - A alternância atualiza temperaturas atuais e previstas sem nova chamada ao forecast; Celsius é a unidade inicial.
  - O fluxo completo permanece acessível e utilizável nos viewports aprovados em T-03.
- **Dependências:** T-11, T-12, T-14.
- **Arquivos prováveis:** `src/App.tsx`, `src/components/`.

## Entrega 7 — Testes após integração

### T-15 — Testar conversão de unidade

- **Tipo:** Test
- **Prioridade:** P1
- **Tamanho:** P
- **Requisitos:** FR-04; AC-02.3, AC-03.4, AC-04.1–AC-04.4
- **Descrição:** Criar testes unitários das funções puras de apresentação da temperatura.
- **Critérios de aceite:**
  - Testes verificam conversão de Celsius para Fahrenheit com valores conhecidos, incluindo 0 °C → 32 °F e 100 °C → 212 °F.
  - Testes verificam Celsius sem conversão, tratamento de `null` e precisão/arredondamento definidos em T-02.
  - Testes não usam DOM, rede ou estado React.
- **Dependências:** T-02, T-05.
- **Arquivos prováveis:** `tests/lib/weather.test.ts`.

### T-16 — Testar services com mock de `fetch`

- **Tipo:** Test
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-01–FR-03, NFR-04; AC-01.1–AC-01.7, AC-02.1–AC-02.5, AC-03.1–AC-03.5
- **Descrição:** Testar geocoding e forecast com respostas de rede mockadas.
- **Critérios de aceite:**
  - Testes verificam URLs/parâmetros e mapeamentos para `City`, `CurrentWeather` e `ForecastDay`.
  - Testes distinguem vazio, HTTP não-2xx, JSON inválido, falha de rede, timeout e resposta parcial.
  - Todos os cenários substituem `fetch`; nenhuma chamada real à Open-Meteo ocorre.
- **Dependências:** T-06, T-07.
- **Arquivos prováveis:** `tests/services/openMeteo.test.ts`.

### T-17 — Testar componentes nos estados de operação

- **Tipo:** Test
- **Prioridade:** P1
- **Tamanho:** G
- **Requisitos:** FR-01–FR-04, NFR-02, NFR-04; AC-01.3–AC-01.7, AC-02.4–AC-02.5, AC-03.3, AC-03.5
- **Descrição:** Testar componentes integrados com Testing Library para loading, erro, vazio e sucesso.
- **Critérios de aceite:**
  - Há assertions distintas para loading, error, empty e success, incluindo vazio de busca e erro de rede/API.
  - Testes cobrem busca/seleção, conteúdo de current/forecast, alternância e campos parciais.
  - Interações essenciais são acionadas por teclado e controles são localizados por role/nome acessível.
- **Dependências:** T-09, T-10, T-11, T-12, T-13.
- **Arquivos prováveis:** `tests/components/`.

### T-18 — Testar hook e transições de estado

- **Tipo:** Test
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-01–FR-04, NFR-04; AC-01.6–AC-01.7, AC-02.2, AC-03.2, AC-04.4
- **Descrição:** Testar orquestração do hook com services mockados.
- **Critérios de aceite:**
  - Testes percorrem `idle`, `loading`, `success`, `empty` e `error` para busca e clima.
  - Geocoding vazio não chama forecast; selecionar cidade chama forecast.
  - Alternar unidade não chama services e resposta antiga não sobrescreve cidade mais recente.
- **Dependências:** T-08.
- **Arquivos prováveis:** `tests/hooks/useWeather.test.ts`.

### T-19 — Testar fluxo E2E principal em desktop e mobile

- **Tipo:** Test
- **Prioridade:** P1
- **Tamanho:** G
- **Requisitos:** FR-01–FR-04, NFR-01, NFR-04, NFR-06; US-01–US-05
- **Descrição:** Cobrir o fluxo principal e cenários de falha com Playwright e APIs interceptadas.
- **Critérios de aceite:**
  - Um teste E2E busca e seleciona cidade, mostra clima atual e previsão de cinco dias e alterna unidade.
  - Testes verificam vazio, erro e resposta parcial sem depender da API real.
  - O fluxo principal conclui no viewport móvel definido em T-03 e não apresenta rolagem horizontal (`scrollWidth <= clientWidth`).
  - A contagem de requests de forecast permanece igual antes/depois da alternância de unidade.
- **Dependências:** T-15, T-16, T-17, T-18, T-21.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`.

## Entrega 8 — Hardening e gates finais

### T-20 — Executar hardening e quality gates

- **Tipo:** Infra
- **Prioridade:** P0
- **Tamanho:** G
- **Requisitos:** FR-01–FR-04, NFR-01–NFR-06
- **Descrição:** Validar release contra critérios aprovados, cenários de resiliência e comandos de qualidade do projeto.
- **Critérios de aceite:**
  - T-01 a T-03 estão registradas na spec e não há decisão bloqueadora pendente para os fluxos implementados.
  - Os fluxos principais e de erro passam nas matrizes de navegador/viewport aprovadas; testes de teclado e acessibilidade definida em T-03 passam.
  - `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e` terminam com exit code `0`.
- **Dependências:** T-19.
- **Arquivos prováveis:** `specs/weather-app-spec.md`, `tests/`, `playwright.config.ts` (somente se ajustes forem necessários).

## Rastreabilidade de Requisitos Funcionais

| Requisito da spec | Tarefas de implementação | Tarefas de validação | Cobertura e pendências |
|---|---|---|---|
| **FR-01 — Busca de cidades** | T-04 (tipos), T-06 (geocoding), T-08 (hook), T-09 (busca/resultados), T-13 (estados), T-14 (integração) | T-16 (service), T-17 (componentes), T-18 (hook), T-19 (E2E), T-20 (hardening) | Coberto. T-01 precisa definir contexto para cidades homônimas e se haverá geolocalização antes de fechar o comportamento. |
| **FR-02 — Clima atual** | T-04 (tipos), T-05 (regras de apresentação), T-07 (forecast), T-08 (hook), T-10 (cartão), T-13 (estados), T-14 (integração) | T-16 (service), T-17 (componentes), T-18 (hook), T-19 (E2E), T-20 (hardening) | Coberto. T-01 precisa definir os campos mínimos; T-02, frescor, fuso e política de timeout/retry. |
| **FR-03 — Previsão de cinco dias** | T-04 (tipos), T-05 (regras de apresentação), T-07 (forecast), T-08 (hook), T-11 (lista), T-13 (estados), T-21 (integração) | T-16 (service), T-17 (componentes), T-18 (hook), T-19 (E2E), T-20 (hardening) | Coberto. T-01 precisa definir granularidade e variáveis; T-02, atualização, fuso e timeout/retry. |
| **FR-04 — Unidade de temperatura** | T-04 (tipos), T-05 (conversão), T-08 (hook), T-10 (cartão), T-11 (previsão), T-12 (controle), T-21 (integração) | T-15 (conversão), T-17 (componentes), T-18 (hook), T-19 (E2E), T-20 (hardening) | Coberto. T-02 precisa definir arredondamento e se a preferência persiste localmente. |

**Lacunas de cobertura:** nenhum requisito funcional da spec está sem tarefa correspondente. A implementação não deve começar até T-01, T-02 e T-03 resolverem as decisões bloqueadoras registradas na spec.

## Sequência sugerida em fatias verticais

A primeira fatia entrega valor visível antes de completar todas as funcionalidades; ela é uma demonstração funcional, não o release completo.

1. **Fundação — sem resultado visual ainda (P0):** T-01 a T-05. Fechar decisões e contratos antes de codificar, conforme as dependências.
2. **Fatia 1 — busca e clima atual (primeiro fluxo visível, P0):** T-06, T-07, T-08, T-09, T-10, T-13 e T-14. Validar os services com T-16 e as transições do hook com T-18. Ao fim, a pessoa consegue buscar uma cidade, selecioná-la e ver o clima atual, com estados de loading, vazio e erro.
3. **Fatia 2 — previsão e unidades (completa os FRs restantes, P1):** T-11, T-12 e T-21. Validar conversão e componentes com T-15 e T-17.
4. **Release:** executar o fluxo completo de desktop/mobile com T-19 e fechar os gates de qualidade com T-20.