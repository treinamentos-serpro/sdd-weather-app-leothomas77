# Plano Técnico — Weather App

## Architecture

Aplicação React de página única com dependências em uma direção: `App` compõe componentes e hook; o hook chama serviços; serviços usam funções puras e contratos. `lib` e `services` não dependem de React, e componentes não acessam a rede diretamente.

1. **Apresentação (`components`)**: busca, resultados, cartões, previsão, alternância e mensagens de estado. Recebe dados e callbacks via props; não contém fetch nem regras de conversão.
2. **Orquestração/estado (`hooks`)**: `useWeather` coordena termo, resultados, cidade selecionada, consultas, unidade e estados `idle/loading/success/error/empty`; entrega à UI um estado coerente e ações.
3. **Acesso a dados (`services`)**: funções assíncronas encapsulam URLs, parâmetros, `fetch`, validação da resposta e mapeamento para os contratos internos. Não conhecem componentes nem renderização.
4. **Funções puras (`lib`)**: conversão de temperatura, descrição de códigos WMO, formatação e transformação determinística dos dados; sem efeitos de rede ou estado.
5. **Contratos (`types`)**: tipos compartilhados para cidade, clima, previsão, unidade e estado de erro, sem implementação de comportamento.

O estado fica no hook usado por `App`; não há necessidade atual de Redux, Context ou cache global. A separação facilita testes isolados: componentes recebem fixtures e callbacks simulados; o hook exercita transições com serviços substituídos; services usam `fetch` mockado; funções de `lib` são testadas diretamente como entradas e saídas puras. Isso reduz setup e torna falhas mais localizáveis.

## Tech Stack

| Tecnologia | Uso e justificativa |
|---|---|
| TypeScript strict | Contratos explícitos para respostas externas e dados normalizados. |
| React 19 + Vite | UI declarativa e servidor/build já adotados pelo repositório. |
| Tailwind CSS | Estilização conforme a stack do projeto. |
| Open-Meteo Geocoding + Forecast | Busca de localidades e dados meteorológicos sem chave de API, conforme a spec. |
| Vitest + Testing Library | Testes unitários e de componentes já previstos pelo projeto. |
| Playwright | Testes de fluxos completos e viewport móvel, já previsto pelo projeto. |
| Biome + pnpm | Lint/formatação e gerenciador de pacotes adotados no repositório. |

Nenhuma dependência adicional de estado, data fetching ou validação é necessária no plano inicial.

## Project Structure

```text
src/
├── components/
│   ├── CitySearch.tsx
│   ├── CityResults.tsx
│   ├── CurrentWeatherCard.tsx
│   ├── ForecastList.tsx
│   ├── UnitToggle.tsx
│   └── WeatherStatus.tsx
├── hooks/
│   └── useWeather.ts
├── services/
│   └── openMeteo.ts
├── lib/
│   └── weather.ts
├── types/
│   └── weather.ts
├── App.tsx
└── main.tsx

tests/
├── lib/weather.test.ts
├── services/openMeteo.test.ts
├── components/
└── e2e/weather.spec.ts
```

- `components/`: um componente por arquivo, conforme a convenção do projeto; apenas apresentação e callbacks.
- `hooks/`: estado/orquestração no limite entre `App`, componentes e serviços.
- `services/`: integração Open-Meteo, sem estado React.
- `lib/`: lógica determinística reutilizável e testável sem DOM/rede.
- `types/`: contratos que impedem divergência entre serviços, hooks e componentes.
- `tests/`: testes Vitest por camada e E2E Playwright, seguindo as configurações existentes.

Essa estrutura mantém componentes pequenos, serviços substituíveis em testes e regras puras independentes da interface, sem introduzir camadas ou bibliotecas além do necessário.

## Data Model

Os dados meteorológicos internos mantêm temperaturas em Celsius e as demais unidades em valores canônicos. A unidade escolhida controla somente a apresentação. Campos retornados como ausentes ou nulos pela API são normalizados para `null`, permitindo respostas parciais sem inventar dados.

```ts
export type Unit = 'celsius' | 'fahrenheit';

export interface City {
  id: number | null; // Identificador da localidade na geocodificação.
  name: string; // Nome da cidade retornado pela busca.
  country: string; // País para distinguir localidades.
  countryCode: string | null; // Código ISO do país, quando disponível.
  region: string | null; // Região administrativa, quando disponível.
  latitude: number; // Latitude usada na consulta meteorológica.
  longitude: number; // Longitude usada na consulta meteorológica.
  timezone: string | null; // Fuso IANA retornado para a localidade, quando disponível.
}

export interface CurrentWeather {
  observedAt: string; // Horário local da observação informado pela API.
  temperatureC: number | null; // Temperatura atual em graus Celsius.
  apparentTemperatureC: number | null; // Sensação térmica em graus Celsius.
  isDay: boolean | null; // Indica dia (true) ou noite (false), a partir de is_day.
  relativeHumidityPercent: number | null; // Umidade relativa em percentual.
  precipitationMm: number | null; // Precipitação atual em milímetros.
  weatherCode: number | null; // Código WMO da condição meteorológica.
  windSpeedKmh: number | null; // Velocidade do vento em km/h.
  windDirectionDegrees: number | null; // Direção do vento em graus.
  pressureHpa: number | null; // Pressão ao nível do mar em hPa.
}

export interface ForecastDay {
  date: string; // Data local no formato ISO YYYY-MM-DD.
  minimumTemperatureC: number | null; // Temperatura mínima em graus Celsius.
  maximumTemperatureC: number | null; // Temperatura máxima em graus Celsius.
  weatherCode: number | null; // Código WMO da condição do dia.
  precipitationMm: number | null; // Precipitação total prevista em milímetros.
  precipitationProbabilityPercent: number | null; // Probabilidade máxima de precipitação em percentual.
  maximumWindSpeedKmh: number | null; // Velocidade máxima do vento em km/h.
}

export interface WeatherData {
  city: City; // Localidade para a qual os dados foram consultados.
  current: CurrentWeather; // Condições atuais normalizadas.
  forecast: ForecastDay[]; // Dias retornados, esperado hoje mais quatro dias.
  timezone: string | null; // Fuso local da resposta, quando informado pela API.
  fetchedAt: string; // Horário ISO da consulta no cliente, para contexto de atualização.
}
```

`weatherCode` é o dado canônico da condição. Um helper puro pode gerar rótulo pt-BR com base na tabela WMO; não armazenar texto localizado vindo de fonte externa. A spec ainda não aprova a lista final de variáveis, então o contrato acima é uma proposta técnica alinhada ao desafio e precisa ser confirmada antes da implementação final.

## Data Flow

```mermaid
flowchart TD
  A[Pessoa digita cidade] --> B[CitySearch]
  B --> C[useWeather: busca loading]
  C --> D[geocoding service]
  D --> E{Resposta de geocoding}
  E -->|Resultados| F[CityResults: estado success]
  E -->|Sem resultados| G[Busca: estado empty]
  E -->|Rede, API ou timeout| H[Busca: estado error]
  F --> I[Pessoa seleciona City]
  I --> J[useWeather: clima loading]
  J --> K[forecast service: current + daily]
  K --> L[Mapeamento e validação em lib/service]
  L --> M{Resposta meteorológica}
  M -->|Válida ou parcial| N[useWeather: estado success]
  M -->|Falha ou timeout| O[useWeather: estado error]
  N --> P[App passa dados e Unit por props]
  P --> Q[CurrentWeatherCard + ForecastList]
  R[UnitToggle] --> S[useWeather: atualiza Unit]
  S --> T[Derivação C/F na renderização]
  T --> Q
```

O estado `empty` é usado quando a busca termina sem cidades; respostas parciais continuam em `success`, com campos ausentes normalizados como `null` e apresentados como indisponíveis.

## External APIs

### Geocoding

- **Base URL:** `https://geocoding-api.open-meteo.com/v1/search`
- **Exemplo de URL:** `https://geocoding-api.open-meteo.com/v1/search?name=Lisboa&count=5&language=pt&format=json`
- **Parâmetros:** `name` é o texto da busca; `count=5` limita resultados; `language=pt` solicita nomes/localizações em português quando suportado; `format=json` explicita o formato.
- **Resposta resumida:**

```json
{
  "results": [
    {
      "id": 2267057,
      "name": "Lisboa",
      "latitude": 38.71667,
      "longitude": -9.13333,
      "country_code": "PT",
      "country": "Portugal",
      "admin1": "Lisboa",
      "timezone": "Europe/Lisbon"
    }
  ]
}
```

- **Mapeamento:** `id`, `name`, `country`, `country_code`, `admin1`, `latitude`, `longitude` e `timezone` mapeiam para `City`. Resposta sem `results` ou com lista vazia normaliza para `[]` e estado `empty`; falha HTTP/rede não deve ser interpretada como ausência de cidade.

### Forecast (condições atuais e previsão diária)

- **Base URL:** `https://api.open-meteo.com/v1/forecast`
- **Exemplo de URL:**

```text
https://api.open-meteo.com/v1/forecast?latitude=38.71667&longitude=-9.13333&current=temperature_2m,relative_humidity_2m,apparent_temperature,precipitation,weather_code,wind_speed_10m,wind_direction_10m,pressure_msl&daily=weather_code,temperature_2m_min,temperature_2m_max,precipitation_sum,precipitation_probability_max,wind_speed_10m_max&temperature_unit=celsius&wind_speed_unit=kmh&precipitation_unit=mm&timezone=auto&forecast_days=5
```

- **Parâmetros relevantes:** `latitude` e `longitude` vêm da `City`; `current` pede campos atuais; `daily` pede campos diários; `timezone=auto` solicita o fuso local inferido pelas coordenadas; `forecast_days=5` solicita hoje e os quatro dias seguintes; os parâmetros de unidade mantêm o contrato canônico em Celsius, km/h e mm.
- **Resposta resumida:**

```json
{
  "timezone": "Europe/Lisbon",
  "current_units": { "time": "iso8601", "temperature_2m": "°C", "wind_speed_10m": "km/h" },
  "current": {
    "time": "2026-10-27T12:15",
    "temperature_2m": 18.4,
    "relative_humidity_2m": 67,
    "apparent_temperature": 18.1,
    "is_day": 1,
    "precipitation": 0,
    "weather_code": 2,
    "wind_speed_10m": 11.2,
    "wind_direction_10m": 240,
    "pressure_msl": 1014.3
  },
  "daily_units": { "time": "iso8601", "temperature_2m_min": "°C", "precipitation_sum": "mm" },
  "daily": {
    "time": ["2026-10-27", "2026-10-28", "2026-10-29", "2026-10-30", "2026-10-31"],
    "weather_code": [2, 3, 61, 2, 1],
    "temperature_2m_min": [14.2, 13.8, 15.0, 14.7, 13.9],
    "temperature_2m_max": [20.1, 19.8, 18.5, 21.0, 20.4],
    "precipitation_sum": [0, 0, 2.4, 0, 0],
    "precipitation_probability_max": [5, 10, 60, 10, 5],
    "wind_speed_10m_max": [17.0, 18.2, 22.1, 15.4, 16.0]
  }
}
```

- **Mapeamento para `City`:** `id`, `name`, `country`, `country_code`, `admin1`, `latitude`, `longitude` e `timezone` de cada resultado mapeiam para os campos homônimos do contrato (`admin1` → `region`). Campos geográficos opcionais viram `null`. `results` ausente ou vazio vira lista vazia e estado `empty`.
- **Mapeamento para `CurrentWeather`:** `current.time` → `observedAt`; `temperature_2m` → `temperatureC`; `apparent_temperature` → `apparentTemperatureC`; `is_day` (0/1) → `isDay` (false/true); `relative_humidity_2m` → `relativeHumidityPercent`; `precipitation` → `precipitationMm`; `weather_code` → `weatherCode`; `wind_speed_10m` → `windSpeedKmh`; `wind_direction_10m` → `windDirectionDegrees`; `pressure_msl` → `pressureHpa`.
- **Mapeamento para `ForecastDay`:** para cada índice `i`, `daily.time[i]` → `date`; `weather_code[i]` → `weatherCode`; `temperature_2m_min[i]`/`temperature_2m_max[i]` → temperaturas mínima/máxima; `precipitation_sum[i]` → `precipitationMm`; `precipitation_probability_max[i]` → `precipitationProbabilityPercent`; `wind_speed_10m_max[i]` → `maximumWindSpeedKmh`.
- **Agregação:** a `City` selecionada, o `current` normalizado, os dias normalizados, `timezone` da resposta Forecast e o horário ISO do cliente em `fetchedAt` formam `WeatherData`. `Unit` é estado de apresentação e não vem da API.
- Validar que `current` exista e que os arrays `daily.*` tenham comprimentos compatíveis. Valores ausentes opcionais viram `null`; resposta estruturalmente inválida é erro, não `empty`. Verificar as unidades recebidas contra os sufixos do contrato.
- **Nota de alinhamento:** `daily`, `timezone=auto`, variáveis e `forecast_days=5` são a proposta deste plano para cumprir o desafio e o período de cinco dias. A spec ainda deixa granularidade, campos e fuso/data como perguntas abertas; aprovar essa proposta exige resolver essas perguntas no produto.

## State Management

- O hook local `useWeather`, montado por `App`, é a única fonte de estado da tela; componentes recebem estado e ações via props. Não adicionar estado global enquanto a aplicação tiver este único fluxo principal.
- Manter dois estados assíncronos independentes, `searchStatus` e `weatherStatus`, ambos com o contrato `idle | loading | success | error | empty`. Isso permite que a busca e o forecast exibam status distintos sem uma máquina global adicional.
- **`idle`:** a operação ainda não começou. **`loading`:** request em andamento. **`success`:** operação concluída com dados válidos; resposta parcial utilizável também fica em `success`. **`empty`:** exclusiva da busca quando geocoding retorna zero cidades. **`error`:** request falhou, expirou ou retornou estrutura inválida.
- `useWeather` mantém em memória termo e resultados da busca, cidade selecionada, `WeatherData`, `Unit` e erro associado à operação. A unidade começa em `celsius`; não há persistência no servidor.
- Dados meteorológicos permanecem em Celsius. Um helper puro deriva valores para exibição durante a renderização: para número `C`, Fahrenheit é $F = C \times 9/5 + 32$; `null` continua indisponível. Min/max atuais e previstos usam o mesmo helper. A troca de unidade não chama geocoding nem forecast.
- Preferência local entre visitas permanece indefinida na spec; até decisão, mantê-la somente em memória, sem `localStorage`.

## Error Handling

- Serviços normalizam falhas em categorias `network`, `http`, `timeout` e `invalid-response`; a camada de UI converte essas categorias em mensagens pt-BR sem expor detalhes técnicos.
- **Rede:** erro de `fetch` define `error` somente para a operação que falhou; manter termo e cidade selecionada para possibilitar nova tentativa.
- **API/HTTP:** verificar `response.ok`; HTTP não-2xx, erro informado pela API e JSON inválido são `error`, nunca `empty`. Não exibir corpo bruto da resposta à pessoa usuária.
- **Timeout:** `AbortController` cancela requests que excedem o limite. O valor exato permanece pendente na spec. Cancelamento intencional por nova busca/seleção não deve ser comunicado como erro.
- **Resposta parcial:** campos meteorológicos opcionais ausentes viram `null`; apresentar os demais campos e indicar a indisponibilidade no campo/dia afetado. Se faltar a estrutura necessária para identificar a cidade, o objeto `current` ou as datas da previsão, tratar como `invalid-response` e `error`.
- **Vazio:** apenas uma resposta de geocoding válida sem cidades define `searchStatus = 'empty'`; não chamar forecast sem seleção. Forecast sem dias utilizáveis é `invalid-response`, não busca vazia.
- **Respostas fora de ordem:** cancelar ou ignorar resultado de request iniciado para termo/cidade anterior para que não sobrescreva a seleção atual.
- **Nova tentativa:** oferecer ação manual explícita para repetir a operação que falhou. Retry automático e número de tentativas ficam fora da decisão inicial até produto definir a política.

## Testing Strategy

### Vitest + Testing Library

- **Funções puras (`lib/weather.ts`):** conversão Celsius/Fahrenheit, preservação de `null`, mapeamento dos códigos WMO e formatação de datas/unidades; cobrir valores de limite e arredondamento depois de decisão de produto.
- **Services:** `fetch` mockado para parâmetros e mapeamento de geocoding/forecast; cobrir resultados vazios, HTTP não-2xx, JSON inválido, campos ausentes, arrays inconsistentes, timeout e cancelamento.
- **Hook:** transições de status, seleção de cidade, mudança de unidade sem nova chamada, erro e descarte de resposta obsoleta.
- **Componentes:** Testing Library para busca, seleção e estados idle/loading/success/error/empty, unidades e informação parcial; verificar labels, teclado e roles acessíveis.
- Testes devem ser determinísticos, sem depender de chamadas reais à Open-Meteo.

### Playwright

- Fluxo feliz: buscar cidade, selecionar resultado, ver clima atual e cinco dias, alternar Celsius/Fahrenheit.
- Fluxos de vazio e falha: cidade sem resultados, geocoding/forecast com erro, timeout e resposta parcial por API interceptada.
- Verificar que alternar unidade não dispara novo request de forecast.
- Rodar fluxo principal em viewport móvel; validar ausência de rolagem horizontal e acesso aos controles principais.
- Não depender de API externa em CI; usar interceptação/fixtures para respostas previsíveis.

## Risks & Trade-offs

| Decisão | Benefício | Trade-off / alternativa considerada |
|---|---|---|
| Um hook local `useWeather` em vez de Redux/Context. | Menos abstração para uma única tela e estado compartilhado pequeno. | Se surgirem muitas rotas ou consumidores independentes, reconsiderar estado global. |
| Um request de Forecast com `current` e `daily`. | Uma coordenada e uma resposta para dados atuais e previsão; facilita consistência. | Campos e granularidade precisam ser aprovados; duas consultas separadas dariam flexibilidade, mas aumentariam coordenação e chamadas. |
| Guardar temperatura em Celsius e derivar unidade na UI. | Toggle imediato e sem novo request; uma fonte canônica para testes. | Requer regras de arredondamento consistentes, ainda em aberto; pedir Fahrenheit à API seria alternativa, mas exigiria request/refetch ao alternar. |
| Manter dados parciais com `null`. | Exibe dados úteis sem inventar valores quando a resposta é incompleta. | Cada componente precisa representar indisponibilidade; alternativa all-or-nothing simplifica a UI, mas perde informação válida. |
| Retry manual inicialmente. | Comportamento explícito e menor risco de carga repetida. | Exige ação da pessoa usuária; retry automático pode melhorar recuperação, mas política não foi decidida. |
| Open-Meteo sem chave. | Setup simples e sem segredo no cliente. | Dependência de disponibilidade, limites e termos do serviço; não há garantia de SLA definida na spec. |
| `timezone=auto` para período diário local. | Datas da previsão correspondem ao fuso inferido da cidade. | A spec ainda precisa confirmar fuso/data e tratamento de exceções de timezone. |

## Revisão P4.8 — Cobertura e Alinhamento

| Requisito da spec | Cobertura no plano |
|---|---|
| FR-01 — Buscar/distinguir/selecionar cidades | Geocoding, `City`, `CityResults`, status vazio/erro e AC-01.1–AC-01.7 na estratégia de testes. |
| FR-02 — Clima atual | `current` no Forecast, `CurrentWeather`, cartão, estados de falha/parcial e testes de service/UI. |
| FR-03 — Cinco dias | `daily` com cinco datas e `forecast_days=5`, `ForecastDay`, lista e testes de arrays/datas. |
| FR-04 — Alternar unidade | `Unit`, dados canônicos Celsius, derivação sem request, testes de conversão e de ausência de novo request. |
| NFR-01/US-05 — Mobile | Viewport móvel em Playwright e estrutura responsiva; matriz de viewports segue pendente. |
| NFR-02 — Acessibilidade | Componentes semânticos, teclado, nomes acessíveis e testes com Testing Library; padrão de conformidade segue pendente. |
| NFR-03 — Desempenho | Estratégia prevê medição por metas, mas thresholds/percentis seguem pendentes; não há alegação de atendimento mensurável ainda. |
| NFR-04 — Resiliência | Estados separados, timeout, respostas parciais, erros e respostas obsoletas definidos. |
| NFR-05 — pt-BR | Textos/localização e códigos WMO com rótulos via helper; formatos de data/hora seguem pendentes. O plano propõe `timezone=auto` para datas diárias, sujeito à confirmação do produto. |
| NFR-06 — Compatibilidade | Testes direcionados são previstos, mas matriz de navegadores/versões segue pendente. |

**Resultado da revisão:** a arquitetura e os contratos cobrem os requisitos funcionais e orientam testes sem introduzir estado global ou novas dependências. Não há contradição com `.github/copilot-instructions.md`: o plano mantém componentes em `src/components/`, serviços em `src/services/`, hooks em `src/hooks/`, tipos em `src/types/`, funções puras e estados loading/error/empty. Antes de declarar o plano totalmente fechado para implementação, resolver na spec os campos e granularidade meteorológicos, contexto dos resultados geográficos, timezone/formato de datas, frescor, timeout/retry, precisão de conversão e metas/matrizes de NFR. Essas pendências são herdadas da spec e estão explicitamente marcadas; não foram decididas por suposição técnica.