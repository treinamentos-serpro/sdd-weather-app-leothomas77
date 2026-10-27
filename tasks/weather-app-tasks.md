# Backlog de Tarefas — Weather App

Tarefas derivadas de `plans/weather-app-plan.md` e rastreadas aos requisitos de `specs/weather-app-spec.md`. As tarefas T-01 a T-03 são bloqueadoras: resolvem perguntas em aberto da spec antes de fixar contratos ou implementar comportamento. Não iniciar tarefas de código até essas decisões serem aprovadas e registradas na spec.

Tipos: **UI** para apresentação/orquestração da interface, **Data** para contratos e integração/regras de dados, **Test** para testes e validação, **Infra** para critérios e configuração compartilhados.

## Entrega 1 — Fechar decisões e contratos

### T-01 — Aprovar escopo e conteúdo meteorológico

- **Tipo:** Data
- **Requisitos:** FR-01, FR-02, FR-03
- **Descrição:** Resolver campos de identificação das cidades, geolocalização opcional, variáveis mínimas de clima atual e granularidade/variáveis da previsão.
- **Critérios de aceite:**
  - A spec define os campos exibidos para distinguir cidades homônimas.
  - A spec confirma se a busca manual é o único fluxo ou se haverá geolocalização.
  - A spec enumera as variáveis mínimas de clima atual e previsão e define se a previsão é diária, horária ou ambas.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-02 — Aprovar políticas de dados e unidades

- **Tipo:** Data
- **Requisitos:** FR-02, FR-03, FR-04, NFR-04, NFR-05
- **Descrição:** Definir frescor dos dados, fuso e formatos de data/hora, timeout/retry, persistência local da unidade e precisão/arredondamento da conversão.
- **Critérios de aceite:**
  - A spec define atualização e idade máxima para dados serem considerados atuais.
  - A spec fixa timeout e política de nova tentativa para geocoding e forecast.
  - A spec define fuso/formatos usados para o período “hoje + quatro dias”.
  - A spec decide persistência local da unidade e regra de precisão/arredondamento.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-03 — Aprovar metas não funcionais

- **Tipo:** Infra
- **Requisitos:** NFR-01, NFR-02, NFR-03, NFR-06
- **Descrição:** Definir matrizes suportadas e metas mensuráveis de responsividade, acessibilidade, compatibilidade e desempenho.
- **Critérios de aceite:**
  - A spec lista viewports/dispositivos móveis e navegadores/sistemas suportados.
  - A spec define o padrão e nível de conformidade de acessibilidade.
  - A spec define metas de latência, percentis e condições de medição.
- **Dependências:** nenhuma.
- **Arquivos prováveis:** `specs/weather-app-spec.md`.

### T-04 — Definir contratos compartilhados

- **Tipo:** Data
- **Requisitos:** FR-01, FR-02, FR-03, FR-04
- **Descrição:** Criar os tipos internos `Unit`, `City`, `CurrentWeather`, `ForecastDay`, `WeatherData` e tipos de status/erro, alinhados às decisões aprovadas.
- **Critérios de aceite:**
  - Os contratos compilam em TypeScript strict e representam os campos decididos na spec.
  - Campos que podem faltar em resposta parcial têm representação explícita de ausência.
  - Os dados meteorológicos mantêm temperatura canônica em Celsius e a unidade de apresentação usa `'celsius' | 'fahrenheit'`.
- **Dependências:** T-01, T-02, T-03.
- **Arquivos prováveis:** `src/types/weather.ts`.

## Entrega 2 — Regras puras e integração de dados

### T-05 — Implementar funções puras de clima

- **Tipo:** Data
- **Requisitos:** FR-02, FR-03, FR-04
- **Descrição:** Implementar conversão de temperatura, mapeamento de códigos WMO e formatação a partir dos contratos aprovados.
- **Critérios de aceite:**
  - Celsius/Fahrenheit são derivados sem efeitos colaterais e `null` permanece indisponível.
  - Códigos WMO suportados produzem rótulos pt-BR conforme decisão aprovada.
  - Formatação respeita as regras de arredondamento e localização definidas na spec.
- **Dependências:** T-02, T-04.
- **Arquivos prováveis:** `src/lib/weather.ts`.

### T-06 — Testar funções puras

- **Tipo:** Test
- **Requisitos:** FR-04, NFR-05
- **Descrição:** Cobrir conversões, valores ausentes, códigos WMO e formatação.
- **Critérios de aceite:**
  - Testes cobrem Celsius→Fahrenheit, Fahrenheit→Celsius, limites e `null`.
  - Testes cobrem códigos meteorológicos mapeados e código desconhecido.
  - Testes cobrem formatos definidos para data, hora e temperatura.
- **Dependências:** T-05.
- **Arquivos prováveis:** `tests/lib/weather.test.ts`.

### T-07 — Implementar serviço de geocoding

- **Tipo:** Data
- **Requisitos:** FR-01
- **Descrição:** Consultar Open-Meteo Geocoding, mapear resultados para `City` e distinguir lista vazia de erro de consulta.
- **Critérios de aceite:**
  - A consulta usa os parâmetros aprovados, codifica o termo de busca e limita resultados conforme decisão.
  - Resultados são normalizados para `City`, incluindo os campos de contexto definidos.
  - Resposta sem resultados retorna coleção vazia; falha HTTP, rede ou JSON inválido retorna erro tipado.
- **Dependências:** T-01, T-04.
- **Arquivos prováveis:** `src/services/openMeteo.ts`.

### T-08 — Testar serviço de geocoding

- **Tipo:** Test
- **Requisitos:** FR-01, NFR-04
- **Descrição:** Testar parâmetros e mapeamento de geocoding usando `fetch` mockado.
- **Critérios de aceite:**
  - Testes verificam URL/parâmetros e mapeamento de cidade.
  - Testes distinguem lista vazia, HTTP não-2xx, JSON inválido, falha de rede e timeout.
  - Nenhum teste depende de chamada real à Open-Meteo.
- **Dependências:** T-07.
- **Arquivos prováveis:** `tests/services/openMeteo.test.ts`.

### T-09 — Implementar serviço de forecast

- **Tipo:** Data
- **Requisitos:** FR-02, FR-03
- **Descrição:** Consultar a API Forecast com campos `current` e `daily` aprovados e mapear respostas para `WeatherData`.
- **Critérios de aceite:**
  - A URL usa latitude/longitude da cidade e parâmetros de current, daily, unidades, fuso e período aprovados.
  - Dados atuais e arrays diários são normalizados para `CurrentWeather` e `ForecastDay` por data.
  - Campos ausentes permitidos viram indisponíveis; resposta estruturalmente inválida retorna erro tipado.
- **Dependências:** T-01, T-02, T-04.
- **Arquivos prováveis:** `src/services/openMeteo.ts`.

### T-10 — Testar serviço de forecast

- **Tipo:** Test
- **Requisitos:** FR-02, FR-03, NFR-04
- **Descrição:** Testar query, mapeamento atual/diário e tratamento de resposta usando `fetch` mockado.
- **Critérios de aceite:**
  - Testes verificam parâmetros, timezone e mapeamento índice-a-data.
  - Testes cobrem resposta parcial, arrays inconsistentes, HTTP não-2xx, JSON inválido, rede e timeout.
  - Não há dependência de rede real.
- **Dependências:** T-09.
- **Arquivos prováveis:** `tests/services/openMeteo.test.ts`.

## Entrega 3 — Orquestração e estado

### T-11 — Implementar hook `useWeather`

- **Tipo:** UI
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-04
- **Descrição:** Orquestrar busca, seleção, consulta, unidade e status sem estado global.
- **Critérios de aceite:**
  - Busca e consulta meteorológica mantêm status independentes `idle/loading/success/error/empty`.
  - Uma cidade selecionada aciona forecast; geocoding vazio não aciona forecast.
  - Trocar unidade recalcula apresentação sem chamar serviços novamente.
  - Resposta atrasada de uma cidade anterior não substitui a seleção atual.
- **Dependências:** T-05, T-07, T-09.
- **Arquivos prováveis:** `src/hooks/useWeather.ts`.

### T-12 — Testar hook e transições de estado

- **Tipo:** Test
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-04
- **Descrição:** Verificar orquestração com serviços mockados.
- **Critérios de aceite:**
  - Testes cobrem idle, loading, success, empty e error para as operações correspondentes.
  - Testes verificam seleção, nova busca, erro/parcial e descarte de resposta obsoleta.
  - Teste verifica que trocar unidade não dispara novo request.
- **Dependências:** T-11.
- **Arquivos prováveis:** `tests/hooks/useWeather.test.ts`.

## Entrega 4 — Interface e integração

### T-13 — Criar busca e lista de cidades

- **Tipo:** UI
- **Requisitos:** FR-01, NFR-02
- **Descrição:** Criar componentes de busca e seleção de resultados recebendo estado e callbacks via props.
- **Critérios de aceite:**
  - A busca permite submeter termo e selecionar cidade.
  - Resultados exibem os campos geográficos aprovados em T-01; selecionar um resultado chama o callback de seleção com aquele `City`.
  - A busca não submete termo vazio ou só com espaços; o campo e cada resultado têm nome acessível e seleção pode ser feita por teclado.
- **Dependências:** T-04.
- **Arquivos prováveis:** `src/components/CitySearch.tsx`, `src/components/CityResults.tsx`.

### T-14 — Criar cartão de clima atual

- **Tipo:** UI
- **Requisitos:** FR-02, FR-04, NFR-01, NFR-02
- **Descrição:** Exibir dados atuais, unidade ativa e campos parcialmente indisponíveis.
- **Critérios de aceite:**
  - Cada campo não nulo do contrato aprovado é exibido para a cidade fornecida por props e temperaturas usam a unidade ativa.
  - Cada campo `null` esperado é marcado como indisponível; nenhum valor substituto é renderizado como dado real.
  - Nos viewports definidos em T-03, os controles e dados do cartão permanecem visíveis e `scrollWidth` da página não excede `clientWidth`.
- **Dependências:** T-04, T-05.
- **Arquivos prováveis:** `src/components/CurrentWeatherCard.tsx`.

### T-15 — Criar lista de previsão

- **Tipo:** UI
- **Requisitos:** FR-03, FR-04, NFR-01, NFR-02
- **Descrição:** Exibir os dias e variáveis da previsão aprovados, incluindo indisponibilidade parcial.
- **Critérios de aceite:**
  - Os dias recebidos são renderizados uma vez cada, em ordem de data local, e temperaturas usam a unidade ativa.
  - Cada campo `null` ou dia incompleto é marcado como indisponível sem valor inventado.
  - Nos viewports definidos em T-03, a lista permanece visível e `scrollWidth` da página não excede `clientWidth`.
- **Dependências:** T-04, T-05.
- **Arquivos prováveis:** `src/components/ForecastList.tsx`.

### T-16 — Criar alternância de unidade

- **Tipo:** UI
- **Requisitos:** FR-04, NFR-02
- **Descrição:** Criar controle acessível para alternar Celsius/Fahrenheit e chamar callback do hook.
- **Critérios de aceite:**
  - O controle expõe a unidade atual e, ao selecionar a outra opção, chama o callback com `'celsius'` ou `'fahrenheit'` uma vez.
  - O controle pode ser focado e acionado por teclado e tem role e nome acessível verificáveis por Testing Library.
  - A alteração não inicia requisição de rede.
- **Dependências:** T-04.
- **Arquivos prováveis:** `src/components/UnitToggle.tsx`.

### T-17 — Criar estados visuais de operação

- **Tipo:** UI
- **Requisitos:** FR-01, FR-02, FR-03, NFR-02, NFR-04
- **Descrição:** Apresentar estados loading, erro, vazio e informação parcial da busca/forecast.
- **Critérios de aceite:**
  - Busca sem resultados renderiza mensagem de vazio diferente da mensagem de erro de rede/API.
  - Erro de current ou forecast identifica a operação afetada e apresenta ação de retry que chama o callback correspondente.
  - Loading é exposto como status acessível; vazio, erro e dados parciais apresentam texto identificável, e nenhum estado fica sem feedback visível.
- **Dependências:** T-04, T-11.
- **Arquivos prováveis:** `src/components/WeatherStatus.tsx`.

### T-18 — Integrar aplicação e responsividade

- **Tipo:** UI
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-01, NFR-05, NFR-06
- **Descrição:** Compor hook e componentes em `App`, aplicar idioma/formatos aprovados e layout responsivo dentro das matrizes definidas.
- **Critérios de aceite:**
  - Com serviços de teste conectados, o fluxo busca → seleção → current/forecast conclui e mostra dados da cidade selecionada; alternar unidade atualiza os valores.
  - Estados `empty` e `error` da busca produzem mensagens distintas; erro de current/forecast identifica a seção.
  - Em cada viewport definido em T-03, busca, dados e unidade permanecem acessíveis e `scrollWidth` não excede `clientWidth`.
- **Dependências:** T-11, T-13, T-14, T-15, T-16, T-17.
- **Arquivos prováveis:** `src/App.tsx`, `src/components/`.

### T-19 — Testar componentes e acessibilidade básica

- **Tipo:** Test
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-01, NFR-02, NFR-04
- **Descrição:** Testar componentes isoladamente com Testing Library nos estados e interações principais.
- **Critérios de aceite:**
  - Há ao menos um teste de interação/assertion para busca e seleção, current, forecast, unidade e dado parcial.
  - Há assertions distintas para loading, error, empty e success nos componentes de estado.
  - Testes acionam busca, seleção e unidade por teclado e localizam controles por role e nome acessível, não apenas por snapshots ou classes CSS.
- **Dependências:** T-13, T-14, T-15, T-16, T-17.
- **Arquivos prováveis:** `tests/components/`.

## Entrega 5 — E2E e qualidade

### T-20 — Criar testes E2E para desktop e mobile

- **Tipo:** Test
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-01, NFR-04, NFR-06
- **Descrição:** Cobrir fluxo principal e falhas com Playwright e respostas de API interceptadas.
- **Critérios de aceite:**
  - Um teste E2E completa busca, seleção, exibição de current e cinco dias e alternância Celsius/Fahrenheit.
  - Testes E2E verificam vazio, erro e resposta parcial com APIs interceptadas; nenhuma chamada depende da Open-Meteo real.
  - No viewport móvel definido em T-03, o fluxo principal conclui e `scrollWidth` não excede `clientWidth`.
  - A contagem de requests de forecast permanece igual antes e depois de alternar unidade.
- **Dependências:** T-18, T-19.
- **Arquivos prováveis:** `tests/e2e/weather.spec.ts`.

### T-21 — Executar gates de qualidade

- **Tipo:** Test
- **Requisitos:** FR-01, FR-02, FR-03, FR-04, NFR-01, NFR-02, NFR-04, NFR-06
- **Descrição:** Rodar os comandos de lint, build, testes unitários e E2E antes de considerar a entrega concluída.
- **Critérios de aceite:**
  - `pnpm lint`, `pnpm build`, `pnpm test` e `pnpm test:e2e` terminam com exit code `0`.
  - A execução final não registra testes falhos nem erros de lint/build.
- **Dependências:** T-06, T-08, T-10, T-12, T-19, T-20.
- **Arquivos prováveis:** `package.json`, `playwright.config.ts`, `tests/` (somente se ajustes forem necessários).