# pouso_modulos_espacial

Sistema de organização da fila e autorização de pouso de módulos espaciais.

## Sobre o projeto

Esse projeto simula parte do gerenciamento de pouso de módulos destinados à Base Aurora, em Marte. Desenvolvido como um protótipo acadêmico do MGPEB na Fase 2 do PBL, ele organiza módulos de habitação, energia, suporte médico, laboratório, logística e mineração, considerando a prioridade de cada módulo e o combustível disponível.

O sistema verifica as condições de clima e a disponibilidade da pista, emitindo mensagens como **“Pouso Prioritário Autorizado”**, **“Pouso Regular Autorizado”**, **“Pouso Negado”** ou uma autorização de emergência por combustível crítico.

O código foi feito em Python, utilizando listas, dicionários, funções definidas com `def`, laços `for` e `while` e estruturas condicionais `if`, `elif` e `else`.

Os dados de cada módulo ficam armazenados em um dicionário. Esses dicionários são adicionados à lista `fila_pouso`, que é reorganizada pelo algoritmo Bubble Sort. A ordenação coloca primeiro os módulos com menor número de prioridade. Quando dois módulos têm a mesma prioridade, aquele com menos combustível é avaliado primeiro.

Depois da ordenação, o programa retira um módulo por vez do início da fila, utilizando `pop(0)`, e chama a função `avaliar_autorizacao_pouso`. Essa função retorna um valor booleano indicando se o pouso foi autorizado e uma mensagem explicando a decisão. Quando a autorização é negada, o módulo é adicionado à lista `pilha_emergencia`, usada como estrutura de contingência.

O código também contém a função `buscar_menor_combustivel`, que localiza o módulo com menos combustível e retorna `None` quando recebe uma lista vazia. Essa função está disponível, mas não é chamada na execução principal.

## Módulos utilizados

| Módulo | Tipo | Prioridade | Combustível no código¹ | Massa (toneladas) |
| --- | --- | ---: | ---: | ---: |
| MOD1 | Habitação | 1 | 18.5 | 18.5 |
| MOD2 | Energia | 1 | 14.2 | 14.2 |
| MOD3 | Suporte médico | 2 | 15.0 | 9.8 |
| MOD4 | Laboratório | 4 | 12.0 | 12.0 |
| MOD5 | Logística | 3 | 22.0 | 22.0 |
| MOD6 | Mineração | 5 | 7.1 | 25.1 |

¹ A saída do programa identifica o combustível com `L`. O relatório utiliza porcentagem, mas o código não faz conversão entre essas unidades. Os valores acima reproduzem o notebook.

Cada módulo também possui os campos `criticidade_carga` e `horario_orbita`. Atualmente, esses campos e a massa são armazenados, mas não participam da ordenação ou da autorização.

## Parâmetros de autorização de pouso

- **Pista livre:** obrigatória para qualquer autorização. Se estiver ocupada, o pouso é negado.
- **Clima favorável:** permite a autorização quando a pista está livre.
- **Combustível crítico:** valor inferior a `10.0`, conforme a implementação atual. Permite uma autorização de emergência com clima desfavorável, desde que a pista esteja livre.
- **Prioridade máxima:** valor igual a `1`. Com clima favorável e pista livre, gera a mensagem de pouso prioritário.
- **Combustível exatamente igual a `10.0`:** não é considerado crítico.

A regra lógica de autorização utiliza as portas AND e OR:

```text
Pouso autorizado = (Clima favorável AND Pista livre)
                  OR (Combustível crítico AND Pista livre)
```

A ordenação e a autorização são etapas separadas. Um módulo com combustível crítico não passa automaticamente à frente dos demais: a prioridade continua sendo o primeiro critério da fila.

## Modelagem matemática

O relatório propõe uma função quadrática para representar a altitude do módulo durante o acionamento dos retrofoguetes. Considerando aceleração constante, o eixo vertical positivo para cima e `v0` como a magnitude da velocidade inicial de descida, a expressão pode ser escrita como:

```text
aceleracao_liquida = aceleracao_retrofoguetes - gravidade_Marte
altura(t) = altura_inicial - velocidade_inicial * t
            + (aceleracao_liquida * t²) / 2
```

- **Altura inicial:** altitude no momento em que os retrofoguetes são acionados, em metros.
- **Velocidade inicial:** magnitude da velocidade vertical descendente, em metros por segundo.
- **Aceleração dos retrofoguetes:** aceleração ascendente, em metros por segundo ao quadrado.
- **Gravidade de Marte:** valor de `3.71 m/s²` adotado no relatório.
- **Tempo:** intervalo desde o acionamento, em segundos.

Quando a aceleração dos retrofoguetes supera a gravidade, a função apresenta concavidade para cima. Essa modelagem faz parte da proposta acadêmica e ainda não está implementada no notebook.

## Tecnologias utilizadas

- **Linguagem:** Python 3.
- **Ambiente:** Google Colab / Jupyter Notebook.
- **Estruturas de dados:** listas e dicionários.
- **Recursos de programação:** funções, laços de repetição e condicionais.
- **Dependências externas:** nenhuma. O código importa `random`, da biblioteca padrão, mas não utiliza essa biblioteca na simulação atual.

## Exemplo da execução

Na configuração inicial, `clima_ok` e `pista_livre` recebem `True`. A fila reorganizada fica nesta ordem:

```text
MOD2 → MOD1 → MOD3 → MOD5 → MOD4 → MOD6
```

O MOD2 vem antes do MOD1 porque ambos possuem prioridade 1, mas o MOD2 tem menos combustível. A saída das autorizações é:

```text
PROCESSAMENTO DE AUTORIZAÇÃO DE POUSO
Processando 'energia': Pouso Prioritário Autorizado.
Processando 'habitação': Pouso Prioritário Autorizado.
Processando 'suporte_medico': Pouso Regular Autorizado.
Processando 'logistica': Pouso Regular Autorizado.
Processando 'laboratorio': Pouso Regular Autorizado.
Processando 'mineração': Pouso Prioritário Autorizado.
```

Com clima desfavorável e pista livre, somente o MOD6 recebe autorização, por ter combustível inferior a `10.0`. Com a pista ocupada, nenhum módulo recebe autorização.

O código foi executado em Python 3.11 durante a preparação deste README. Foram conferidos a ordem da fila, a saída inicial, as oito combinações de clima, pista e combustível crítico, os limites de combustível e a busca em uma lista vazia.

## Como executar

### Via Google Colab

1. Acesse o [notebook do projeto](https://colab.research.google.com/drive/1wn5xnEEqYZyFeN1oLxZ7GvCVvOv0XMQI?usp=sharing).
2. Se quiser alterar os dados, salve uma cópia no seu Google Drive.
3. Conecte um ambiente de execução Python, caso seja solicitado.
4. Execute a célula pelo botão de execução e acompanhe os resultados abaixo dela.

Não é necessário instalar bibliotecas ou utilizar GPU. Para repetir a simulação, execute a célula inteira novamente, pois a fila é esvaziada durante o processamento.

### Via VS Code ou Jupyter Lab/Notebook

O código está disponível no Colab e ainda não foi incluído neste repositório. Para executar localmente:

1. Tenha Python 3 e um ambiente com suporte a notebooks. No VS Code, utilize as extensões Python e Jupyter.
2. Clone o repositório:

```bash
git clone https://github.com/devpalavicini/pouso_modulos_espacial.git
cd pouso_modulos_espacial
```

3. Baixe o notebook em formato `.ipynb` pelo Colab e salve-o na pasta do projeto. O relatório o identifica como `Fase2Projeto.ipynb`.
4. Abra o notebook no VS Code ou no Jupyter, selecione um kernel Python e execute a célula de código.

Também é possível copiar o conteúdo completo da célula para um arquivo `Fase2Projeto.py` e executá-lo pelo terminal:

```bash
python Fase2Projeto.py
```

## Próximos passos

- Padronizar a unidade do combustível e alinhar o código ao relatório, que propõe combustível nominal a partir de 15% e emergência abaixo de 5%.
- Incluir o notebook no GitHub para permitir a execução diretamente a partir do repositório.
- Implementar a verificação de sensores, o registro dos módulos que concluíram o pouso e o reprocessamento dos módulos em contingência.
- Revisar os horários dos módulos e definir se eles devem participar da organização da fila.
- Implementar a função matemática e o gráfico de altitude.
- Completar a contextualização sobre a evolução da computação e os princípios ESG, considerando consumo de recursos, segurança da tripulação e registro das decisões.

O protótipo atual avalia autorizações, sem simular a trajetória física, o consumo de combustível ou a ocupação da pista ao longo do tempo.

## Material do projeto

- [Código no Google Colab](https://colab.research.google.com/drive/1wn5xnEEqYZyFeN1oLxZ7GvCVvOv0XMQI?usp=sharing).
- [Relatório PBL — Fase 2](https://docs.google.com/document/d/135kLGPIWkxqu8W2TtbFUvnoLXs_3QqKoPBIHMV4L9y4/edit?tab=t.0).