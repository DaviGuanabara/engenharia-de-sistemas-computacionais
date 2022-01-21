## Rotatória e roteador: Fowarding function

### Introdução.

#### Analogia

Imagine uma via de veículos com pedágio. Suas entradas direcionam todos os carros de diferentes origens para um mesmo pedágio. Durante o pedágio, os carros aguardam pacientemente em uma fila a execução de uma série de operações, como o pagamento da taxa de uso da via. Após essas operações, os carros atravessam a via de transporte encaminhando-se até as suas respectivas saídas.

O *data plane* de um roteador funciona de forma análoga a essa via imaginada. Os *datagrams*, análogos aos carros, após entrarem no roteador (pelos *inputs*), são enfileirados e ficam no aguardo de serem processados. Após uma série de operações, os mesmos são direcionados para as suas respectivas sáidas (*outputs*).

Assim, o roteador é um caso específico da abstração mais genérica *match plus action* (corresponder e agir), o qual é performado por outros dispositivos, como os *switches*, e não apenas pelos roteadores.

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


### Input Port

A Figura 02 mostra as execuções ocorridas na *input port*. Inicia-se com a ação de funções relativas a *physical layer* (em *Line Termination*), seguida de funções da *link layer* (em *Data link processing*), por fim o *lookup, fowarding* e *queuing* (em *lookup, fowarding, queuing*). 

Figura 02: Execuções na Input Port\
![Image](imagens/input%20port.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 314.

A função central da porta de entrada é o *lookup*, no qual o roteador procura (*look up*) na *forwarding table* qual porta o *datagram* recém chegado deve ser direcionado.
Apesar da proeminente importância do *lookup*, outras ações também devem ser tomadas, como a chacagem do *version number*, *checksum* e *Time to Live*

Como pode-se supor, em uma arquitetura de banco de dados centralizado, os registros da *forwarding table* estariam disponíveis em um único local do roteador, com a operação de *look up* devendo fazer uma requisição de registro à esse banco de dados, algo que gera um possível gargalo afinal, caso o módulo do banco de dados não conseguir suprir a demanda das requisições, deverão ocorrer filas e atrasos. 

Para evitar esse gargalo, cada linha da tabela é copiada para cada um dos *input ports* presentes no roteador, de forma que o acesso à tabela de transmissão (*forwarding table*) seja feita localmente (no *input port*), sem a necessidade de requisições.

No *look up*, o roteador identifica a saída (*link interface*) utilizando a regra de maior correspondência (*longest prefix matching rule*) de prefixo do IP *Address* de destino do *datagram*. Um exemplo dessas correspondências podem ser vistas na Tabela 01.
(O prefixo é formado por x primeiros dígitos do IP *Address* de destino referencia o endereço da subrede.)

*Tabela 1 Exemplo de forwarding table*

Prefix | Link Interface 
-----------|-------------
11001000 00010111 00010       | 0  
11001000 00010111 00011000    | 1  
11001000 00010111 00011       | 2    
otherwise                     | 3       

Por exemplo, o endereço de IP:

´´´
11001000 00010111 00011000 10101010
´´´

Tem o prefixo correspondendo ao *link interface* 1 e 2, com o *link* 1 sendo a maior correspondência e, portanto, sendo os dados direcionados ao mesmo.



### Switching


O *switching fabric* tem a função de conectar as *input ports* com as *output ports*. Há diferentes formas de implementá-lo, mas três delas, mostradas na Figura 02, destacam-se:


Figura 02: Tipos de Switching\
![Image](imagens/switching.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 317.


1. *Switching via memory*: o *input port* sinaliza, a partir de uma interrupção, para o processador do roteador a chegada de novos *datagrams*. Após a interrupçãos, os dados recém chegados são copiados para a memória, processados (determinando-se a interface apropriada), e por fim copiados para o *buffer* de saída. Isso é usualmente feito com processos com memória compartilhada.Como desvantagem pode-se citar, primeiro, a impossibilidade de transmissão de mais de um *datagram* ao mesmo tempo, pois somente uma operação de leitura e escrita na memória pode ser feita por vez. Segundo, com a largura de banda sendo B *datagrams* por segundo para memória (B *datagrams* podem ser escritos ou lidos em 1 segundo), significa dizer que a taxa de transferência está limitada em B/2 (pois será necessário fazer 1 operação de escrita e uma de leitura).
2. *Switching via bus*: a *input port* envia os *datagrams* diretamente para a sua respectiva *output port* através de um barramento compartilhado, sem a intervenção do processador do roteador. Isso é usualmente feito agregando-se um *header* com um rótulo informando sua respectiva interface. Ao ser transmitido, todas as interfaces recebem esses dados, porém somente aquela indicada pelo rótulo irá manter o mesmo, retirando o seu rótulo, processando-o, e transmitindo-o para fora do roteador. O primeiro fato incoveniente dessa arquitetura é que somente 1 pacote de dados podem pecorrer o barramento. Isso significa que se chegarem multiplos *datagrams* em diferentes *input ports*, todos menos 1 devem esperar o barramento tornar-se disponível. O segundo problema é que a velocidade do roteador estará limitada pela velocidade do barramento barramento. Esse tipo de arquitetura é indicado para LAN.


..................


Várias outras vias anexam-se a ela  é delimitada por um pedágio, e o seu "corpo" conecta-se com  


Um roteador é form
Um roteador é formado por 3 componentes, as portas de entrada, o comutador, e as portas de saída. De forma análoga, um roteador pode ser visto como uma via de carros, com uma rotatória no centro.


1. Porta de entrada (*input port*) (vias de entradas de carros): análogo ás vias de entrada de carros, é responsável por executar funções relacionadas as camadas físicas e enlace, com a função mais crucial sendo a de *lookup* (a determinação da porta de saída)
