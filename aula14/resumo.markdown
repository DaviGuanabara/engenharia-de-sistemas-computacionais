## Rotatória e roteador: Fowarding function

### Introdução.

#### Analogia

Imagine uma via de veículos com pedágio. Suas entradas direcionam todos os carros de diferentes origens para um mesmo pedágio. Durante o pedágio, os carros aguardam pacientemente em uma fila a execução de uma série de operações, como o pagamento da taxa de uso da via. Após essas operações, os carros atravessam a via de transporte encaminhando-se até as suas respectivas saídas.

O *data plane* de um roteador funciona de forma análoga a essa via imaginada. Os *datagrams*, análogos aos carros, após entrarem no roteador (pelos *inputs*), são enfileirados e ficam no aguardo de serem processados. Após uma série de operações, os mesmos são direcionados para as suas respectivas sáidas (*outputs*).

A Figura 01 apresenta a arquitetura de um roteador, separada em *control plane* e *data plane*, que implementam o *routing* e o *fowarding, respectivamente*. 

Figura 01: Arquitetura de um roteador\
![Image](imagens/Roteador.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 311.

#### Descrição

De forma mais descritiva, o roteador é composto por:

1. *input ports*: armazena os *datagrams* recém chegados e performa funções da camada física e de enlace, como o *lookup*, a determinação da porta de saída (para tal, é necessário consultar a *fowarding table*).
2. *Switching fabric*: conecta os *input ports* com os *Output ports*.
3. *Output ports*: armazena os *datagrams* recebidos pela *Switching fabric* e transmite-os para fora do roteador (performando funções da camada física e de enlace essenciais).
4. *Routing processor*: performa funções do *control plane*, como computar as rotas e manter a *fowarding table* (em roteadores SDN, as operações mencionadas são feitas remotamente).

É importante mencionar que os pontos 1, 2 e 3 são quase sempre implementados em *hardware*.

#### Tipos de forwarding

O *output link* pode ser determinada de duas formas:

1. *Destination-based fowarding*: no qual a saída é baseada no destino.
2. *Generalized forwarding*: em que o *output link* é definido com base em N fatores, como a origem dos dados.

Voltando para a analogia, o *Destination-based fowarding* seria o atendente do pedágio direcionar o carro a uma saída específica baseado no destino que o motorista almeja. No caso do *generalized forwarding*, fatores como o tipo do veículo pode impactar na orientação dada pelo atendente.


#### Questões a a cerca do forwarding.


E se:
1. O atende for capaz de atender 1 carro por minuto, mas chegarem 2 carros por minuto ?
2. Todos os carros que entrarem quiserem ir para a mesma saída ?
3. Tiver mais carro entrando do que saindo ?
4. For necessário tornar prioritário alguns veículos (como ônibus) e bloquear a entrada de outros (como caminhões acima de um certo peso) ?



Para as perguntas 1, 2 e 3, fica claro que gerará tráfego na entrada, na saída e na via, respectivamente. Por fim, a resposta da pergunta 4 passa pela criação de regras prévias a partir da definição das políticas de filtro.

O mesmo pode ocorrer em um roteador, com a chegada de múltiplos *datagrams* em uma mesma *input port* sendo maior que sua capacidade de processá-los podendo gerar filas, atrasos e perda de dados (com o mesmo ocorrendo na saída). A quantidade limitada de *datagrams* suportada pelo *switch fabric* pode não suportar a alta demanda de fluxo de dados, causando o bloqueio de novos entrantes e, consequentemente, filas e atrasos.

Essas questões serão debatidas posteriormente em mais detalhes.

..................


Várias outras vias anexam-se a ela  é delimitada por um pedágio, e o seu "corpo" conecta-se com  


Um roteador é form
Um roteador é formado por 3 componentes, as portas de entrada, o comutador, e as portas de saída. De forma análoga, um roteador pode ser visto como uma via de carros, com uma rotatória no centro.


1. Porta de entrada (*input port*) (vias de entradas de carros): análogo ás vias de entrada de carros, é responsável por executar funções relacionadas as camadas físicas e enlace, com a função mais crucial sendo a de *lookup* (a determinação da porta de saída)
