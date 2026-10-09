# pouso_modulos_espacial

## Módulo de Gerenciamento de Pouso e Estabilização de Base — MGPEB

Protótipo acadêmico de organização da fila e autorização de pouso de módulos destinados à Base Aurora, em Marte, desenvolvido para a Fase 2 do PBL da missão Aurora Siger.

Esta documentação descreve a implementação corrigida do MGPEB com os dados do relatório da Fase 2. Para reproduzir os resultados apresentados, o cadastro do código deve corresponder à tabela deste README.

## Sobre o projeto

O sistema cadastra os módulos, organiza a fila, realiza buscas, avalia as condições de pouso e registra os módulos pousados, os módulos em espera e as ocorrências de alerta.

As decisões são baseadas em expressões booleanas que representam as portas lógicas AND e OR do relatório, implementadas com condicionais `if`, `elif` e `else`.

A simulação é organizada em rodadas independentes. Todos os módulos são considerados presentes em órbita no início da rodada. Cada pouso autorizado é tratado como concluído antes da avaliação do próximo módulo, sem simular o tempo de descida ou a ocupação da pista durante a manobra.

A modelagem matemática, a contextualização histórica da computação e a reflexão sobre ESG complementam o projeto no relatório técnico. O programa não implementa toda a operação física ou a governança da colônia.

## Módulos utilizados

O combustível é representado em porcentagem, a massa em toneladas e os horários no formato `HH:MM`. Um número menor no campo `prioridade` representa maior prioridade operacional.

| Módulo | Tipo | Prioridade | Combustível (%) | Massa (t) | Criticidade da carga | Horário em órbita | Sensores íntegros |
| --- | --- | ---: | ---: | ---: | --- | --- | --- |
| MOD1 | Habitação | 1 | 19.0 | 18.5 | Alta | 14:30 | `True` |
| MOD2 | Energia | 1 | 12.1 | 14.2 | Alta | 14:35 | `True` |
| MOD3 | Suporte médico | 2 | 22.0 | 9.8 | Alta | 14:40 | `True` |
| MOD4 | Laboratório científico | 4 | 18.9 | 12.0 | Média | 14:45 | `True` |
| MOD5 | Logística e suprimentos | 3 | 20.0 | 22.0 | Média | 14:50 | `True` |
| MOD6 | Mineração e extração | 5 | 7.1 | 25.1 | Baixa | 14:55 | `True` |

Cada módulo é representado por um dicionário com os campos `id_modulo`, `tipo`, `prioridade`, `combustivel`, `massa_ton`, `criticidade_carga`, `horario_orbita` e `sensores_ok`.

A massa e a criticidade são armazenadas para descrever o cenário, mas não participam das regras atuais de ordenação ou autorização. O horário organiza a fila inicial; a prioridade e o combustível determinam sua reorganização para atendimento.

## Funcionamento da simulação

A função `simular_pousos` valida o cadastro e coloca cópias dos módulos na `fila_pouso`, preservando os dados originais. A fila é organizada primeiro pelo horário de chegada. Na execução com exibição habilitada, são demonstradas as buscas pelo menor combustível, pela maior prioridade e pelo tipo de carga.

O Bubble Sort reorganiza a fila por prioridade, usando o menor combustível como desempate. Empates completos preservam a ordem anterior. Em seguida, o programa retira um módulo por vez com `pop(0)` e chama `avaliar_autorizacao_pouso`, que retorna um booleano e o motivo da decisão.

Os pousos considerados concluídos são registrados na `lista_base`; os adiados, na `lista_espera`. Os alertas são registrados separadamente e lidos em ordem LIFO ao final da rodada.

**A ordem da fila não garante autorização.** Combustível crítico também não concede precedência automática sobre módulos de maior prioridade na ordenação adotada.

## Parâmetros de autorização de pouso

As condições booleanas correspondem às expressões apresentadas no relatório:

| Símbolo | Condição | Representação no código |
| --- | --- | --- |
| C | Combustível suficiente para pouso nominal | `modulo["combustivel"] >= 15.0` |
| A | Atmosfera adequada | `clima_favoravel` |
| D | Área de pouso disponível | `pista_livre` |
| S | Sensores íntegros | `modulo["sensores_ok"]` |
| E | Emergência por combustível crítico | `modulo["combustivel"] < 5.0` |

```text
Pnominal = C AND A AND D AND S
Pemergencia = E AND D
Pautorizado = Pnominal OR Pemergencia
```

O pouso nominal exige todas as condições do primeiro ramo. Se ele não for possível, o sistema verifica a regra de emergência. Quando nenhum ramo permite a autorização, o pouso é adiado e os motivos são registrados.

Para dados válidos, os limites são interpretados assim:

| Combustível | Consequência na regra |
| --- | --- |
| De 0% até menos de 5% | Ativa a condição de emergência; exige pista livre. |
| De 5% até menos de 15% | Não atende ao mínimo nominal nem à condição de emergência. |
| De 15% até 100% | Atende ao requisito de combustível nominal; ainda depende de clima, pista e sensores. |

Com a pista ocupada, nenhum pouso é autorizado.

> **Limitação da regra de emergência:** `E AND D` é uma simplificação acadêmica. Esse ramo não exige clima favorável nem sensores íntegros e também aceita combustível igual a zero. A autorização lógica não demonstra a viabilidade física ou a segurança da manobra. O protótipo não deve ser utilizado como sistema real de controle de pouso.

### Validação dos dados

Antes da rodada, são verificados os campos obrigatórios, os identificadores repetidos, a prioridade inteira positiva, o combustível entre 0 e 100%, a massa positiva e finita, o horário válido e os valores booleanos de sensores, clima e pista. Dados inválidos interrompem a rodada com `ValueError`.

## Estruturas de dados

As filas e pilhas são implementadas com listas de Python. O comportamento de cada estrutura depende das operações utilizadas para inserir e retirar seus elementos.

| Estrutura | Finalidade |
| --- | --- |
| `lista_modulos` | Cadastro dos módulos e seus atributos. |
| `fila_pouso` | Módulos que ainda serão avaliados na rodada; utiliza `append()` e `pop(0)`. |
| `lista_base` | Módulos cujo pouso foi autorizado e considerado concluído. |
| `lista_espera` | Módulos avaliados que tiveram o pouso adiado. |
| `lista_alerta` | Módulos com ocorrências de alerta na rodada. |
| `pilha_emergencia` | Registros de alerta com identificador e motivo; utiliza `append()` e `pop()`. |
| `alertas_processados` | Registros preservados depois da leitura da pilha. |
| `historico_decisoes` | Identificador, autorização e motivo de cada decisão da rodada. |

### Fila: primeiro a entrar, primeiro a sair

Após a reorganização, a fila é processada do início para o fim. A chamada `fila_pouso.pop(0)` retira o primeiro módulo dessa sequência.

No cenário documentado:

```text
Fila inicial:       MOD1 → MOD2 → MOD3 → MOD4 → MOD5 → MOD6
Fila reorganizada:  MOD2 → MOD1 → MOD3 → MOD5 → MOD4 → MOD6
Após retirar MOD2:  MOD1 → MOD3 → MOD5 → MOD4 → MOD6
```

O comportamento FIFO vale para a sequência reorganizada; a ordenação por prioridade modifica a ordem cronológica inicial.

### Listas de acompanhamento

Com clima favorável, pista livre e os dados da tabela, a `lista_base` recebe MOD1, MOD3, MOD5 e MOD4, nessa ordem. A `lista_espera` recebe MOD2 e MOD6.

Um módulo pode estar em espera e também na `lista_alerta`. Essa sobreposição não representa módulos adicionais: uma lista indica o resultado operacional e a outra identifica ocorrências que exigem atenção.

### Pilha: último a entrar, primeiro a sair

Os alertas são empilhados durante o processamento. A função `processar_alertas` retira o último registro usando `pop()`.

```text
Ordem de inserção dos alertas: MOD2 → MOD6
Topo da pilha:                 MOD6
Ordem de retirada:             MOD6 → MOD2
```

No código, combustível abaixo de 15%, sensores falhos ou clima desfavorável geram registros de alerta. Pista ocupada, isoladamente, gera espera, mas não um registro nessa pilha.

A leitura esvazia a `pilha_emergencia` e preserva seus registros em `alertas_processados`. **Ler um alerta não resolve sua causa.** A `lista_alerta` permanece disponível como registro das ocorrências da rodada.

## Algoritmos de busca e ordenação

As funções `buscar_menor_combustivel` e `buscar_maior_prioridade` realizam buscas lineares. Ambas retornam `None` para uma lista vazia e mantêm o primeiro encontrado em caso de empate.

A função `buscar_tipo_carga` retorna todos os módulos cujo campo `tipo` corresponde ao texto informado, desconsiderando maiúsculas, minúsculas e espaços nas extremidades. Sem correspondências, retorna uma lista vazia.

Na demonstração inicial, a busca pelo menor combustível encontra MOD6. A busca pela maior prioridade encontra MOD1, pois ele aparece antes de MOD2 na fila organizada por horário. A busca por `"energia"` encontra MOD2. **As buscas não alteram a fila.**

A ordenação é realizada por `ordenar_fila_pouso`, que implementa o Bubble Sort. Com `por_horario=True`, compara os horários. Na chamada padrão, compara a prioridade e utiliza o combustível como desempate.

## Exemplo da execução

Para reproduzir este cenário, utilize os dados da tabela, todos os sensores com `True`, clima favorável e pista livre:

```python
resultado = simular_pousos(
    lista_modulos,
    clima_ok=True,
    pista_livre=True
)
```

A ordem de avaliação será:

```text
MOD2 → MOD1 → MOD3 → MOD5 → MOD4 → MOD6
```

MOD2 fica antes de MOD1 porque ambos têm prioridade 1, mas MOD2 possui menos combustível. O trecho das decisões será:

```text
PROCESSAMENTO DE AUTORIZAÇÃO DE POUSO
MOD2: Pouso adiado: combustível abaixo do mínimo nominal de 15%.
MOD1: Pouso nominal autorizado.
MOD3: Pouso nominal autorizado.
MOD5: Pouso nominal autorizado.
MOD4: Pouso nominal autorizado.
MOD6: Pouso adiado: combustível abaixo do mínimo nominal de 15%.
```

| Resultado da rodada | Módulos |
| --- | --- |
| Pousados, na ordem de registro | MOD1, MOD3, MOD5 e MOD4 |
| Em espera | MOD2 e MOD6 |
| Com ocorrências de alerta | MOD2 e MOD6 |
| Alertas lidos em ordem LIFO | MOD6 e MOD2 |

```text
Resumo: 4 pousados; 2 em espera; 2 com alertas.
```

MOD2 e MOD6 ficam em espera porque seus combustíveis estão entre 5% e 15%. Eles não atendem ao mínimo nominal nem à faixa de emergência. Nenhum dos seis módulos ativa a emergência por combustível crítico no cadastro documentado.

Com clima desfavorável, os mesmos seis módulos ficam em espera, pois nenhum possui combustível abaixo de 5%. Com a pista ocupada, todos também ficam em espera.

### Exemplos complementares

A função `demonstrar_cenarios` utiliza cópias de um módulo, sem alterar o cadastro principal. Ela apresenta pouso nominal com 20% de combustível; adiamentos por pista ocupada, clima desfavorável, falha nos sensores ou combustível intermediário de 12%; e uma emergência com 3% de combustível, clima desfavorável e pista livre.

Nesse último caso, a autorização decorre da simplificação `E AND D`, e não de uma avaliação física de segurança.

## Modelagem matemática

O relatório utiliza uma função quadrática para representar a altura durante a frenagem pelos retrofoguetes. O modelo considera aceleração constante, sentido positivo para cima e `v0` como a magnitude positiva da velocidade inicial de descida.

```text
a_liquida = a_retro - g_Marte
h(t) = h0 - v0*t + (a_liquida*t²)/2
v(t) = -v0 + a_liquida*t
```

Nessas expressões, `h0` é a altura inicial em metros; `v0`, a magnitude da velocidade inicial de descida em m/s; `a_retro`, a aceleração ascendente dos retrofoguetes; `g_Marte`, a gravidade adotada no relatório, de 3,71 m/s²; e `t`, o tempo em segundos. A diferença entre as acelerações é representada por `a_liquida`.

No exemplo do relatório, `h0 = 100 m`, `v0 = 20 m/s` e `a_liquida = 2 m/s²`:

```text
h(t) = 100 - 20t + t²
v(t) = -20 + 2t

h(10) = 100 - 20(10) + 10² = 0 m
v(10) = -20 + 2(10) = 0 m/s
```

| Tempo (s) | Altura estimada (m) |
| ---: | ---: |
| 0 | 100 |
| 2 | 64 |
| 4 | 36 |
| 6 | 16 |
| 8 | 4 |
| 10 | 0 |

A análise considera apenas `0 ≤ t ≤ 10 s`. Nesse intervalo, a altura diminui e a velocidade de descida é reduzida até o contato com o solo. A continuação da parábola após 10 segundos não representa a operação de pouso considerada.

Uma velocidade inicial maior exige mais distância de frenagem. Uma aceleração líquida de frenagem maior reduz a distância necessária, mantidas as demais condições. A modelagem auxilia a discutir a altitude de acionamento dos retrofoguetes e os limites do pouso idealizado.

**Essa modelagem é apresentada no relatório; o código de autorização não calcula a trajetória nem gera o gráfico automaticamente.**

## Contextualização histórica e ESG

O relatório relaciona os primeiros computadores de propósito geral, como o ENIAC, à evolução para transistores, circuitos integrados e sistemas embarcados. Discute limitações de memória, processamento, energia e tolerância à radiação que influenciariam um sistema destinado a Marte.

O Bubble Sort e as listas permitem acompanhar as operações dos seis módulos. Entretanto, o crescimento das comparações e o deslocamento dos elementos causado por `pop(0)` precisariam ser avaliados em uma aplicação embarcada com restrições de recursos e prazos.

A reflexão ESG aborda a escolha da área de pouso, preservação científica, prevenção de contaminação, gestão de energia, água e resíduos, uso responsável de recursos locais, saúde, participação dos moradores e transparência nas decisões.

Essas diretrizes fazem parte da concepção da base. O histórico de decisões registra uma rodada em memória, mas não constitui um sistema completo de auditoria, governança ou gestão ambiental.

## Tecnologias utilizadas

O protótipo utiliza **Python 3**, listas, dicionários, funções, laços, condicionais e expressões booleanas. Pode ser executado pelo terminal, pelo Google Colab ou em um ambiente compatível com Jupyter Notebook.

**Não há dependências externas no script.** O programa não utiliza `random`, não depende de serviços externos e não precisa de GPU.

## Como executar

### Via Google Colab

1. Abra o [notebook indicado no relatório](https://colab.research.google.com/drive/1IV4PP6u4ChXEOuPfbhXlQ-3iIC9uegde?usp=sharing) ou carregue o arquivo `.ipynb` corrigido.
2. Confira o cadastro e execute as células de código em ordem.
3. Acompanhe a fila, as buscas, as decisões, os alertas e o resumo apresentados na saída.

Cada chamada a `simular_pousos` cria novas estruturas de acompanhamento. É possível repetir a execução sem reaproveitar a fila consumida da rodada anterior. O cadastro original não é alterado pela função.

### Via terminal ou VS Code

Salve o código corrigido em um arquivo chamado `Fase2Projeto.py`. No terminal, dentro da pasta desse arquivo, execute:

```bash
python Fase2Projeto.py
```

Ao executar o arquivo completo, o programa realiza a rodada principal e apresenta os exemplos complementares. Para utilizar o formato `.ipynb`, abra o notebook em um ambiente compatível com Jupyter e execute as células de código.

### Alterar as condições da rodada

O clima e a disponibilidade da pista são parâmetros gerais da rodada. Os sensores e o combustível são atributos de cada módulo.

```python
# Avalia o cadastro com clima desfavorável e pista livre.
resultado = simular_pousos(
    lista_modulos,
    clima_ok=False,
    pista_livre=True
)
```

Esse trecho deve ser executado depois das definições do cadastro e das funções. Os resultados retornados podem ser acessados, por exemplo, em `resultado["lista_base"]`, `resultado["lista_espera"]` e `resultado["historico_decisoes"]`.

## Limitações do protótipo

A simulação não calcula consumo de combustível, passagem do tempo, chegadas progressivas, trajetória real, ocupação temporizada da pista ou colisões. As condições de clima e pista permanecem fixas durante cada rodada.

Os módulos em espera não são reavaliados automaticamente. Uma nova chamada representa outra rodada independente, e a escolha dos módulos que participarão dela deve ser feita explicitamente. O sistema não mantém o estado operacional entre rodadas.

Os registros permanecem em memória, sem banco de dados ou salvamento automático. O processamento da pilha apenas lê e preserva alertas, sem corrigir falhas físicas ou alterar a ordem de pouso.

O objetivo é demonstrar os conteúdos estudados na atividade, não fornecer um software certificado de navegação, segurança ou controle espacial.

## Material do projeto

- [Repositório do projeto](https://github.com/devpalavicini/pouso_modulos_espacial).
- [Google Colab indicado no relatório](https://colab.research.google.com/drive/1IV4PP6u4ChXEOuPfbhXlQ-3iIC9uegde?usp=sharing).
- [Relatório PBL — Fase 2](https://docs.google.com/document/d/135kLGPIWkxqu8W2TtbFUvnoLXs_3QqKoPBIHMV4L9y4/edit?tab=t.0).
