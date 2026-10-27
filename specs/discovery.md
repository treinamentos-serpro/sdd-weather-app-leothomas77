# Discovery — Weather App

## Contexto

A empresa solicitou uma aplicação de previsão do tempo. O briefing identifica quatro capacidades: buscar cidades, consultar o clima atual, ver a previsão de cinco dias e alternar entre Celsius e Fahrenheit. Também exige que a aplicação possa ser usada em dispositivos móveis.

O briefing não define fonte de dados, detalhes da previsão, critérios de qualidade nem vários comportamentos de busca e apresentação. As decisões adotadas para este treinamento estão registradas na seção **Decisões**; os demais pontos que podem alterar escopo ou experiência permanecem em **Perguntas em Aberto**.

## Requisitos Funcionais

- **RF-01 — Buscar cidades:** permitir que a pessoa usuária pesquise uma cidade e selecione o local para consultar.
- **RF-02 — Consultar clima atual:** apresentar as condições meteorológicas atuais da cidade selecionada.
- **RF-03 — Consultar previsão:** apresentar a previsão do tempo para cinco dias da cidade selecionada.
- **RF-04 — Alternar unidade de temperatura:** permitir alternar a temperatura exibida entre Celsius e Fahrenheit.

## Requisitos Não-Funcionais

- **RNF-01 — Responsividade:** busca, clima atual, previsão e alternância de unidade devem permanecer utilizáveis em dispositivos móveis, conforme solicitado no briefing. Dimensões e dispositivos-alvo ainda precisam ser definidos.
- **RNF-02 — Acessibilidade:** os controles e informações devem ser utilizáveis por pessoas com diferentes necessidades, incluindo navegação por teclado e tecnologias assistivas. O nível de conformidade ainda precisa ser definido.
- **RNF-03 — Desempenho:** busca e consulta meteorológica devem responder em tempo adequado para a pessoa usuária. Metas de tempo ainda precisam ser acordadas.
- **RNF-04 — Resiliência:** a aplicação deve informar quando uma busca não retorna resultados ou quando uma consulta não pode ser concluída. O comportamento detalhado para falhas ainda precisa ser definido.
- **RNF-05 — Localização:** a interface deve ser apresentada em pt-BR, conforme decisão do treinamento. Formatos de data, hora e demais unidades precisam ser confirmados quando aplicável.

## Riscos

| Risco | Probabilidade | Impacto | Mitigação |
|---|---|---|---|
| A fonte meteorológica ficar indisponível, lenta ou limitar consultas. | Média | Alto | Avaliar disponibilidade e limites da fonte; definir feedback para falhas e tempos de espera. |
| A busca retornar cidades homônimas ou uma localização incorreta. | Média | Alto | Identificar resultados com contexto geográfico suficiente, como região e país; validar a seleção. |
| O significado de “previsão de cinco dias” ser interpretado de formas diferentes. | Alta | Alto | Adotar para o treinamento hoje mais os quatro dias seguintes e confirmar o nível de detalhe esperado. |
| A experiência apresentar problemas em certos dispositivos móveis. | Média | Alto | Definir dispositivos e larguras de tela-alvo e validar as tarefas principais nesses contextos. |
| A alternância de unidade produzir valores ou rótulos inconsistentes. | Baixa | Médio | Definir claramente o que é convertido e verificar que valores e rótulos exibidos correspondam à unidade escolhida. |
| Falhas de rede deixarem a pessoa sem explicação ou sem estado visível. | Média | Alto | Definir mensagens e estados para carregamento, erro, timeout e ausência de resultados. |

## Perguntas em Aberto (Open Questions)

1. **Como diferenciar cidades com o mesmo nome nos resultados da busca?**
	- **Impacto:** sem região ou país, a pessoa pode consultar o clima do local errado.
2. **A aplicação deve oferecer localização automática ou a busca manual é o único fluxo?**
	- **Impacto:** afeta o primeiro uso, as permissões solicitadas e o caminho para consultar o clima.
3. **A previsão deve ser diária, horária ou incluir os dois níveis?**
	- **Impacto:** muda a quantidade de informação e a forma como a previsão será consumida.
4. **Com que frequência os dados meteorológicos devem ser atualizados?**
	- **Impacto:** afeta a atualidade da informação e o volume de consultas à fonte de dados.
5. **O que deve ser exibido quando não houver conexão ou a fonte de dados falhar?**
	- **Impacto:** sem uma resposta definida, a pessoa pode não saber se deve aguardar, tentar novamente ou escolher outra cidade.
6. **A preferência Celsius/Fahrenheit deve ser mantida após recarregar ou retornar à aplicação?**
	- **Impacto:** afeta a consistência da experiência entre visitas; a decisão de não persistir no servidor não esclarece se a preferência pode ser mantida localmente.
7. **Quais navegadores, dispositivos e larguras de tela devem ser suportados?**
	- **Impacto:** sem alvos definidos, não é possível validar compatibilidade e responsividade objetivamente.
8. **Qual nível ou padrão de acessibilidade deve ser atendido?**
	- **Impacto:** sem um critério, acessibilidade fica difícil de verificar de forma consistente.
9. **Quais formatos de data e hora devem ser usados na interface pt-BR?**
	- **Impacto:** formatos inconsistentes podem confundir a leitura da previsão.

## Suposições (Assumptions)

- A pessoa usuária escolhe uma cidade pela busca antes de consultar clima e previsão.
- Clima atual e previsão correspondem à mesma cidade selecionada.
- A alternância Celsius/Fahrenheit aplica-se aos valores de temperatura; outras unidades meteorológicas não foram especificadas.
- O requisito de uso em dispositivos móveis implica que as quatro capacidades funcionais devem estar disponíveis também em telas móveis.
- As decisões listadas abaixo são premissas do treinamento e não necessariamente requisitos universais para qualquer produto meteorológico.

## Personas

- **Pessoa que decide o que vestir:** consulta o clima atual para se preparar para sair.
- **Pessoa que planeja a semana:** consulta a previsão para organizar atividades e compromissos.
- **Pessoa viajante:** busca cidades de destino para se preparar antes ou durante uma viagem.

## Decisões

Para destravar a especificação neste treinamento, ficam estabelecidas as decisões a seguir:

- **Fonte de dados:** Open-Meteo, sem chave de API.
- **Período da previsão:** cinco dias significam hoje e os quatro dias seguintes.
- **Unidade padrão:** Celsius.
- **Autenticação e persistência:** sem autenticação e sem persistência de dados no servidor.
- **Idioma da interface:** pt-BR.
