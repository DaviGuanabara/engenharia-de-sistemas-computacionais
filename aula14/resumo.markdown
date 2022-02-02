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
1. O atendente for capaz de atender 1 carro por minuto, mas chegarem 2 carros por minuto ?
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
3. *Switching via an interconnection network*: também chamado de cruzamento de barras, esse tipo de switch, como mostrado na Figura 02, pode alternar cada cruzamento de barra (ou nó) entre aberto e fechado de forma independente. Dessa maneira, multiplos *datagrams* podem atravessar em paralelo o *switch fabric* ao mesmo (ou seja, *non-blocking* para uma *output port* específica). Por exemplo, para emitir um dado do *input* A para o *output* Y, basta fechar o cruzamento dos mesmos. Esse fechamento não impedirá que os dados do *input* B alcance a saída X, porém o *output* Y estará somente disponível para o *input* A (estando bloqueado para os outros *inputs*). 


### Output port


A Figura 03 mostra as 3 principais execuções do *output port*. Inicia-se com o enfileiramento (*Queuing*) dos *datagrams* recebidos. Em seguida, são performados ações necessárias relativas as camadas físicas e enlace (*Data link processing*). Por fim, os dados são enviados para fora do roteador (*Line termination*).


Figura 03: Output Port\
![Image](imagens/output%20port.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 319.


### Filas

As filas podem se formar na entrada e na saída do roteador.

As filas na entrada podem ser formadas pela diferença de velocidade entre o *input* e o *switching fabric* (como citado anteriormente, de forma análoga, na questão 1 do "E se" no tópico" Questões a a cerca do forwarding"). Se um *input* for capaz de lidar com uma taxa de *datagrams* mais alta do que o *switching fabric*, os mesmos ficarão acumulados na entrada até executados pelo *switching fabric*.

Um outro evento que têm como consequência a geração de filas é o chamado *head-of-the-line blocking* (HOL *blocking*). 
A Figura 03 mostra um exemplo de como o HOL *blocking* pode ocorrer. Os *datagrams* azuis-escuros estão destinados às saídas superiores, enquanto que os azuis-claros estão destinados às saídas centrais. Assim, um azul-escuro oriundo do *input* superior pode bloquear a passagem do *datagram* azuil-escuro oriundo do *input* inferior, travando a fila e impedindo que o *datagram* azul-claro seja processado paralelamente, apesar de sua saída estar livre.


Figura 03: Hol Blocking\
![Image](imagens/hol%20blocking.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 321.


Na saída, as filas podem se formar quando multiplos *datagrams* dos *inputs* são direcionados para o mesmo *output*, como mostrado na Figura 04. Esse evento pode preencher o *buffer* de saída, ocasionando a "derrubada" de novos *datagrams*, política chamada de *drop-tail*, ou a remoção de um já enfileirado, para assim criar espaço para os dados recém chegados. Em alguns casos, pode ser vantajoso remover um pacote de dados antes que fila fique cheia, de forma a enviar um sinal de congestionamento para o emissor. Os algoritmos responsáveis por isso são chamados de *Active Queue Management* (AQM), ou gerenciador de filas ativo, com o *Random Early Detection* (RED) sendo um algoritmo dessa classe amplamente implementado.
O *buffer* de saída po

Figura 04: Fila na saída\
![Image](imagens/output%20queue.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 322.


O tamanho do espaço destinado para as filas sofrem de uma dicotomia. Enquanto que *buffers* pequenos podem não apresentar espaço suficiente para lidar com picos de demanda, algo que aumenta o número de dados perdidos, filas grandes podem representar um longo tempo de espera, aumentando o atraso na entrega dos *datagrams*.

Para equilibrar esses diferentes pontos, o tamanho do *buffer* (`B`) fora relacionado com o *round-trip time* (RTT) médio (período entre a emissão dos dados e a recepção do seu ACK) e a capacidade do link (`C`).


```
B = RTT . C

B = 2.5G bits, para RTT = 250 ms, e C = 10 Gbps
```


Atualmente, também relaciona-se o número de fluxos independentes de TCP (`N`):

```
B = RTT . C / √N
```

#### Prioridades dos dados

Na discurssão dos *buffers* fora deixado implícito a política *First-in-First-Out* (FIFO), ou primeiro a chegar, primeiro a sair (modelo mostrado na Figura 05), mas outras regras também podem ser utilizadas, como a classificação dos dados em diferentes filas a partir de sua prioridade (modelo mostrado na Figura 06).


Figura 05: Modelo FIFO\
![Image](imagens/FIFO.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 325.

Figura 06: Modelo classificação e priorização\
![Image](imagens/priority%20queue.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 326.

Uma forma generalizada de classificação e priorização dos dados que tem sido bastante implementada é o chamado *Weighted Fair Queuing* (WFQ), no qual as filas de maior peso são tratadas primeiro, seguindo para as de menor peso, até reiniciar o ciclo, como mostrado na Figura 07.


Figura 07: Weighted Fair Queuing\
![Image](imagens/WFQ.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 326.

##### Neutralidade das redes



Como qualquer regra pode ser imposta para a classificação dos dados, os ISP's podem não ser neutros no oferecimento de seus serviços, algo que vem resultando leis que regulamentam do que os ISP's podem ou não fazer.

Em geral, a defesa da neutralidade das redes pode ser resumido em 3 pontos:

1. Conteúdos não devem ser bloqueados (*No Blocking*).
2. Tráfico de Internet não deve ser penalizado (*No Throttling*).
3. Não deve haver priorização paga (*Paid Prioritization*).


### Generalized Forwarding

O *Generalized Forwarding* é a forma genérica do paradigma *match plus action*, no qual utiliza os *headers* associados aos protocolos de diferentes camadas para determinar o que deve ser feito com os dados recebidos. Para a sua implementação, é utilizado o padrão OpenFlow, o qual basea-se no SDN (*Software Defined Network*), que aplica um controlador remoto para o cálculo das regras da rede.

A base do *Generalized Forwarding* está na *Flow Table*, tabela no qual está contido as regras do *match plus action*. 
Cada regra têm:

1. Um conjunto de *headers* à serem verificados.
2. Um conjunto de contadores.
3. Um conjunto de ações à serem tomadas.


Perceba que essa tabela é uma forma limitada de programação. Atualmente, vem ganhando força uma alternativa mais rica de possibilidades de implementação (com variáveis e funções), a linguagem *Programming Protocol-independent Packet Processors* (P4).


Dentro do padrão OpenFlow, os *headers* podem ser oriundos das camadas de enlace, rede e transporte, como mostrado na Figura 08. É importante perceber que nem todos os *headers* podem ser escolhidos para a ação de correspondência (*match*) (assim fora implementado como uma forma de equilibrar a funcionalidade e complexidade. É melhor fazer algo simples e bem, do que muita coisa e ruim).

Figura 08: OpenFlow Headers\
![Image](imagens/Open%20Flow%20headers.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 356.

As ações a serem tomadas são:

1. *Forwarding*: a transmissão dos dados.
2. *Dropping*: o "deixar de lado".
3. *Modify-field*: a modificação de algum *header*.





