# Especificação de Produto — Weather App

## Overview

O Weather App permite que pessoas consultem informações meteorológicas de cidades escolhidas por busca. O produto apresenta o clima atual, a previsão para cinco dias e temperaturas em Celsius ou Fahrenheit. As funções principais devem ser utilizáveis em dispositivos móveis.

### Objetivos

- Encontrar e selecionar uma cidade para consulta.
- Consultar condições meteorológicas atuais e a previsão da cidade selecionada.
- Alternar a unidade de temperatura entre Celsius e Fahrenheit.
- Realizar essas tarefas em dispositivos móveis.

### Decisões confirmadas

- Fonte meteorológica: Open-Meteo, sem chave de API.
- Período da previsão: hoje e os quatro dias seguintes.
- Unidade inicial: Celsius.
- Idioma da interface: pt-BR.
- Sem autenticação e sem persistência de dados no servidor.

## Functional Requirements

- **FR-01 — Busca de cidades:** a pessoa usuária pode pesquisar cidades por texto, distinguir resultados e selecionar uma cidade para consultar.
- **FR-02 — Clima atual:** para a cidade selecionada, o produto apresenta os dados atuais disponíveis da fonte meteorológica.
- **FR-03 — Previsão de cinco dias:** para a cidade selecionada, o produto apresenta a previsão abrangendo hoje e os quatro dias seguintes.
- **FR-04 — Unidade de temperatura:** a pessoa usuária pode alternar os valores de temperatura exibidos entre Celsius e Fahrenheit; Celsius é a unidade inicial.

## User Stories

- **US-01 (FR-01) — Encontrar uma cidade:** Como pessoa viajante, quero buscar e selecionar uma cidade para consultar as condições meteorológicas do meu destino.
- **US-02 (FR-02) — Consultar o clima atual:** Como pessoa que decide o que vestir, quero ver o clima atual da cidade selecionada para me preparar para sair.
- **US-03 (FR-03) — Planejar os próximos dias:** Como pessoa que planeja a semana, quero consultar a previsão para hoje e os quatro dias seguintes para organizar atividades e compromissos.
- **US-04 (FR-04) — Escolher a unidade de temperatura:** Como pessoa que decide o que vestir, quero alternar entre Celsius e Fahrenheit para interpretar a temperatura na unidade que prefiro.
- **US-05 (FR-02, FR-03; NFR-01) — Consultar o tempo em dispositivo móvel:** Como pessoa viajante, quero consultar o clima atual e a previsão da cidade selecionada no celular para me preparar durante o deslocamento.

## Acceptance Criteria

Todos os critérios abaixo usam **Dado / Quando / Então** (Given / When / Then). Os nomes dos campos meteorológicos, limites de desempenho e outros pontos ainda não decididos estão listados em **Open Questions** e impedem a aprovação dos critérios dependentes.

### US-01 / FR-01 — Encontrar uma cidade

- **AC-01.1:** **Dado** que a pessoa informou um termo de busca não vazio, **quando** confirmar a busca e a geocodificação retornar correspondências, **então** o produto apresenta resultados selecionáveis relacionados ao termo.
- **AC-01.2:** **Dado** que a geocodificação retorne cidades homônimas, **quando** os resultados forem apresentados, **então** cada resultado inclui contexto geográfico suficiente para distingui-lo dos demais; os campos exatos ainda precisam ser decididos.
- **AC-01.3:** **Dado** que a pessoa informe uma cidade inexistente ou sem correspondência, **quando** a geocodificação terminar sem resultados, **então** o produto informa que nenhuma cidade foi encontrada, não consulta o clima e permite corrigir a busca.
- **AC-01.4:** **Dado** que o campo esteja vazio ou contenha somente espaços, **quando** a pessoa confirmar a busca, **então** o produto solicita um termo válido e não inicia a geocodificação.
- **AC-01.5:** **Dado** que o termo contenha espaços, acentos, hífen ou apóstrofo, **quando** a pessoa buscar, **então** o texto é tratado sem erro e os resultados correspondem ao termo processado pela fonte; se não houver correspondência, aplica-se AC-01.3.
- **AC-01.6:** **Dado** que resultados selecionáveis estejam visíveis, **quando** a pessoa selecionar um deles, **então** o produto usa a cidade selecionada para as consultas de clima atual e previsão.
- **AC-01.7:** **Dado** que o serviço de geocodificação falhe, **quando** a pessoa buscar, **então** o produto informa uma falha de consulta distinta de “nenhuma cidade encontrada” e mantém a busca disponível.

### US-02 / FR-02 — Consultar o clima atual

- **AC-02.1:** **Dado** que uma cidade esteja selecionada e a consulta de clima atual retorne dados válidos, **quando** o produto concluir a consulta, **então** apresenta os dados recebidos associados à cidade selecionada.
- **AC-02.2:** **Dado** que uma cidade esteja selecionada, **quando** a pessoa selecionar outra cidade, **então** os dados atuais passam a corresponder somente à nova seleção e não são apresentados como pertencentes à cidade anterior.
- **AC-02.3:** **Dado** que a unidade ativa seja Celsius ou Fahrenheit, **quando** o clima atual for exibido, **então** todo valor de temperatura visível usa essa unidade e tem rótulo correspondente.
- **AC-02.4:** **Dado** que a resposta de clima atual esteja parcial, **quando** o produto apresentar o resultado, **então** mostra os dados disponíveis e identifica os campos ausentes como indisponíveis, sem inventar valores.
- **AC-02.5:** **Dado** que a consulta de clima atual falhe ou expire, **quando** o produto encerrar a consulta, **então** mostra um estado de erro para o clima atual sem representar dados antigos como atuais.

### US-03 / FR-03 — Planejar os próximos dias

- **AC-03.1:** **Dado** que uma cidade esteja selecionada e a previsão retorne dados válidos, **quando** o produto apresentar o período, **então** ele abrange hoje e os quatro dias seguintes, identificando as cinco datas sem duplicação ou omissão.
- **AC-03.2:** **Dado** que uma data do período tenha dados de previsão, **quando** a previsão for apresentada, **então** os dados são associados à data e à cidade selecionadas.
- **AC-03.3:** **Dado** que a resposta da previsão esteja parcial, **quando** faltar dado para uma data ou campo, **então** o produto apresenta os dados disponíveis e indica a parte indisponível sem fabricar uma previsão.
- **AC-03.4:** **Dado** que a unidade ativa seja Celsius ou Fahrenheit, **quando** a previsão exibir temperaturas, **então** todos os valores visíveis usam essa unidade e têm rótulos correspondentes.
- **AC-03.5:** **Dado** que a consulta da previsão falhe ou expire, **quando** o produto encerrar a consulta, **então** mostra um estado de erro para a previsão sem representar dados antigos como atuais.

### US-04 / FR-04 — Escolher a unidade de temperatura

- **AC-04.1:** **Dado** que não exista preferência de unidade aplicada, **quando** o produto iniciar, **então** temperaturas são exibidas em Celsius com rótulo correspondente.
- **AC-04.2:** **Dado** que temperaturas atuais ou previsão estejam visíveis em Celsius, **quando** a pessoa selecionar Fahrenheit, **então** todos os valores visíveis são convertidos para Fahrenheit e seus rótulos são atualizados.
- **AC-04.3:** **Dado** que temperaturas estejam visíveis em Fahrenheit, **quando** a pessoa selecionar Celsius, **então** todos os valores visíveis são convertidos para Celsius e seus rótulos são atualizados.
- **AC-04.4:** **Dado** que a pessoa altere a unidade, **quando** os valores forem atualizados, **então** a cidade selecionada e as condições meteorológicas não relacionadas à temperatura permanecem inalteradas.

### US-05 / FR-02, FR-03 / NFR-01 — Consultar o tempo em dispositivo móvel

- **AC-05.1:** **Dado** um dispositivo e viewport incluídos na matriz de suporte móvel, **quando** a pessoa buscar uma cidade, consultar clima atual e previsão e alternar a unidade, **então** consegue concluir essas tarefas sem conteúdo ou controles essenciais ocultos.
- **AC-05.2:** **Dado** um viewport móvel suportado, **quando** o conteúdo principal for apresentado, **então** textos e controles permanecem legíveis e operáveis sem rolagem horizontal da página.
- **AC-05.3:** Os dispositivos e dimensões que compõem a matriz de suporte precisam ser definidos antes da validação final deste critério.

## Non-Functional Requirements

- **NFR-01 — Responsividade:** busca, clima atual, previsão e alternância de unidade devem ser utilizáveis na matriz móvel aprovada. A matriz de dispositivos e viewports permanece pendente.
- **NFR-02 — Acessibilidade:** busca, resultados, alternância de unidade e mensagens de estado devem ser operáveis por teclado, ter nome acessível e apresentar foco perceptível. O padrão de conformidade e a validação com tecnologias assistivas ainda precisam ser definidos.
- **NFR-03 — Desempenho:** busca, clima atual e previsão devem cumprir metas de latência definidas para condições e percentis acordados. Os limites e condições de medição ainda precisam ser definidos.
- **NFR-04 — Resiliência:** falhas, timeouts, resultados vazios e respostas parciais devem ser diferenciados; dados ausentes ou antigos não podem ser apresentados como dados atuais válidos.
- **NFR-05 — Localização:** textos da interface devem estar em pt-BR. Formatos de data, hora e fuso horário ainda precisam ser definidos.
- **NFR-06 — Compatibilidade:** o suporte a navegadores, sistemas operacionais e versões deve ser definido em uma matriz antes de liberar o produto.

## Edge Cases

- **Cidade inexistente:** informar que nenhuma cidade correspondente foi encontrada, não consultar clima e permitir editar o termo.
- **Input vazio ou somente espaços:** não iniciar geocodificação; solicitar um termo de busca.
- **Caracteres especiais:** aceitar termos com espaços, acentos, hífens e apóstrofos sem erro; mostrar resultados correspondentes ou o estado sem resultados.
- **Geocoding sem resultados:** apresentar estado vazio distinto de falha de API; não iniciar consulta meteorológica até a seleção de uma cidade.
- **Falha da API de geocodificação:** informar que a busca não pôde ser concluída, sem afirmar que a cidade inexiste.
- **Falha da API meteorológica:** mostrar erro na seção afetada, manter a cidade selecionada e não apresentar dados antigos como atuais.
- **Timeout:** encerrar o estado de carregamento e informar que a consulta expirou; o limite de tempo e o fluxo de nova tentativa precisam ser definidos.
- **Resposta parcial:** apresentar cada seção ou data com dados válidos e identificar dados ausentes como indisponíveis; nunca preencher lacunas com valores inventados.
- **Resposta atrasada de uma busca anterior:** se a pessoa selecionar uma nova cidade antes de uma consulta anterior terminar, não substituir os dados da nova seleção por uma resposta antiga.
- **Cidades homônimas:** apresentar contexto suficiente para seleção inequívoca; os campos usados para esse contexto ainda precisam ser decididos.

## Assumptions

- A pessoa seleciona uma cidade por busca; localização automática não é presumida.
- Clima atual e previsão correspondem à cidade selecionada.
- A alternância Celsius/Fahrenheit aplica-se aos valores de temperatura; outras unidades meteorológicas não estão especificadas.
- “Cinco dias” significa hoje e os quatro dias seguintes.
- Open-Meteo é a fonte de dados escolhida para este treinamento e não requer chave de API.
- A interface é em pt-BR, a unidade inicial é Celsius e não há autenticação nem persistência no servidor.
- Não se presume persistência local da unidade preferida.

## Risks

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| A fonte meteorológica ficar indisponível, lenta ou limitar consultas. | Média | Alto | Definir limites de timeout e política de nova tentativa; informar falhas sem apresentar dados antigos como atuais. |
| A busca retornar cidades homônimas ou localização incorreta. | Média | Alto | Definir os campos de contexto geográfico e validar a distinção dos resultados. |
| O nível de detalhe da previsão não atender à necessidade das personas. | Alta | Alto | Decidir granularidade e variáveis antes de aprovar tarefas e testes da previsão. |
| A experiência apresentar problemas em dispositivos móveis ou navegadores relevantes. | Média | Alto | Aprovar matrizes de dispositivos, viewports e navegadores e validar os fluxos principais nelas. |
| Valores convertidos ou rótulos de unidade ficarem inconsistentes. | Baixa | Médio | Definir precisão/arredondamento e verificar a conversão em todos os dados de temperatura visíveis. |
| Dados parciais ou desatualizados serem confundidos com dados completos e atuais. | Média | Alto | Identificar indisponibilidade e atualidade dos dados; nunca preencher lacunas nem rotular dados antigos como atuais. |

## Out of Scope

- Contas, autenticação e perfis de usuário.
- Persistência de dados no servidor.
- Favoritos, histórico de cidades e sincronização entre dispositivos, não solicitados no briefing.
- Alertas meteorológicos, notificações, mapas e radar.
- Operação offline e garantia de dados em cache.
- Geolocalização automática não faz parte do escopo confirmado; sua inclusão depende da decisão registrada em **Open Questions**.

## Open Questions

1. **Quais campos geográficos devem identificar cada resultado da busca (por exemplo, região e país)?**
   - **Impacto:** sem esses campos, cidades homônimas não podem ser distinguidas de modo consistente.
2. **A aplicação deve oferecer localização automática ou manter apenas a busca manual?**
   - **Impacto:** altera o fluxo inicial, permissões e critérios de aceite.
3. **A previsão deve ser diária, horária ou ambas, e quais variáveis deve apresentar?**
   - **Impacto:** define conteúdo, layout e critérios testáveis da previsão.
4. **Quais variáveis devem compor o clima atual?**
   - **Impacto:** AC-02.1 não pode ser validado de forma completa enquanto não houver conjunto mínimo de dados definido.
5. **Com que frequência os dados devem ser atualizados e qual idade máxima ainda pode ser apresentada como atual?**
   - **Impacto:** define frescor, cache e validade dos dados mostrados.
6. **Qual deve ser o limite de timeout para geocodificação, clima atual e previsão?**
   - **Impacto:** sem limite numérico, o comportamento de timeout não é reproduzível em testes.
7. **Em caso de falha, a nova tentativa deve ser manual, automática ou ambas? Quantas tentativas são permitidas?**
   - **Impacto:** define recuperação, latência percebida e carga sobre a fonte.
8. **A preferência Celsius/Fahrenheit deve persistir localmente entre visitas?**
   - **Impacto:** altera o comportamento ao iniciar sem preferência e os critérios de unidade.
9. **Qual precisão e regra de arredondamento devem ser usadas na conversão e exibição de temperaturas?**
   - **Impacto:** necessário para verificar valores convertidos e evitar diferenças entre telas.
10. **Quais dispositivos, larguras de viewport e navegadores compõem as matrizes móvel e de compatibilidade?**
    - **Impacto:** sem alvos definidos, responsividade e compatibilidade não podem ser aprovadas objetivamente.
11. **Qual padrão e nível de conformidade de acessibilidade devem ser atendidos?**
    - **Impacto:** impede uma validação consistente de acessibilidade e tecnologias assistivas.
12. **Quais formatos de data/hora e qual fuso horário devem ser usados para cada cidade?**
    - **Impacto:** pode alterar a data exibida para “hoje” e a interpretação da previsão.
13. **Quais metas de latência, percentis e condições de rede devem ser usados para busca e consultas meteorológicas?**
    - **Impacto:** NFR-03 não é mensurável até que os valores e o método de medição sejam definidos.

## Revisão Crítica Aplicada

| Problema identificado | Correção aplicada ou decisão pendente |
|---|---|
| Critérios anteriores usavam descrições genéricas sem condições reproduzíveis. | Reescritos em Given/When/Then com IDs e resultados observáveis. |
| Busca não cobria cidade inexistente, input vazio, caracteres especiais e geocoding sem resultados. | Adicionados critérios e comportamentos separados para cada caso. |
| Falhas e respostas parciais não distinguiam erro, ausência de resultado e dados incompletos. | Definido feedback por seção, preservação da cidade selecionada e proibição de inventar ou rotular dados antigos como atuais. |
| Campos de clima, granularidade da previsão, precisão de unidade e metas de NFR não estavam definidos. | Registrados como perguntas abertas, pois escolher valores sem decisão de produto seria especulativo. |
| História móvel e seus critérios não tinham alvos de validação nem rastreabilidade completa. | Ligada a FR-02/FR-03 e NFR-01; matriz de dispositivos permanece pendente. |

## Readiness Assessment

**A especificação ainda não é suficiente para desenvolver e validar o produto sem novas perguntas.** O escopo central está definido, mas as seguintes decisões bloqueiam critérios finais e testes de aceitação completos:

- Campos geográficos apresentados nos resultados de busca.
- Inclusão ou não de geolocalização automática.
- Variáveis mínimas do clima atual e da previsão e granularidade diária/horária.
- Atualização/frescor dos dados e idade máxima para considerá-los atuais.
- Limite de timeout e política de novas tentativas.
- Persistência local da unidade e precisão/arredondamento das conversões.
- Matrizes de dispositivos, viewports e navegadores suportados.
- Padrão de acessibilidade e metas quantitativas de desempenho.
- Formatos de data/hora e fuso horário usado para determinar “hoje”.

O **Out of Scope** explicita as exclusões confirmadas ou não solicitadas. Geolocalização automática permanece pendente e não deve ser tratada como exclusão definitiva até que a pergunta correspondente seja decidida.

## Traceability

| User Story | Requisitos funcionais | Critérios de aceite | Requisitos não funcionais relevantes |
|---|---|---|---|
| US-01 — Encontrar uma cidade | FR-01 | AC-01.1 a AC-01.7 | NFR-02, NFR-03, NFR-04, NFR-05, NFR-06 |
| US-02 — Consultar o clima atual | FR-02 | AC-02.1 a AC-02.5 | NFR-02, NFR-03, NFR-04, NFR-05, NFR-06 |
| US-03 — Planejar os próximos dias | FR-03 | AC-03.1 a AC-03.5 | NFR-02, NFR-03, NFR-04, NFR-05, NFR-06 |
| US-04 — Escolher a unidade de temperatura | FR-04 | AC-04.1 a AC-04.4 | NFR-02, NFR-05, NFR-06 |
| US-05 — Consultar o tempo em dispositivo móvel | FR-02, FR-03 | AC-05.1 a AC-05.3 | NFR-01, NFR-02, NFR-04, NFR-05, NFR-06 |