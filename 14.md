## Rotatória e roteador: Fowarding function

### Introdução.

#### Analogia

Imagine uma via de veículos com pedágio. Suas entradas direcionam todos os carros de diferentes origens para um mesmo pedágio. Durante o pedágio, os carros aguardam pacientemente em uma fila a execução de uma série de operações, como o pagamento da taxa de uso da via. Após essas operações, os carros atravessam a via de transporte encaminhando-se até as suas respectivas saídas.

O *data plane* de um roteador funciona de forma análoga a essa via imaginada. Os *datagrams*, análogos aos carros, após entrarem no roteador (pelos *inputs*), são enfileirados e ficam no aguardo de serem processados. Após uma série de operações, os mesmos são direcionados para as suas respectivas sáidas (*outputs*).

Assim, o roteador é um caso específico da abstração mais genérica *match plus action* (corresponder e agir), o qual é performado também por outros dispositivos, como os *switches*, e não apenas pelos roteadores.

A Figura 01 apresenta a arquitetura de um roteador, separada em *control plane* e *data plane*, que implementam o *routing* e o *fowarding, respectivamente*. 


| ![Image](imagens/Roteador.png)|
|:--------:|
|<b>Figura 01: Arquitetura de um roteador</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 311.</b>|


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
1. O atendente somente for capaz de atender 1 carro por minuto, mas chegarem 2 carros por minuto ?
2. Todos os carros que entrarem forem orientados para a mesma saída ?
3. Tiver mais carro entrando do que saindo ?
4. For necessário tornar prioritário alguns veículos (como ônibus) e bloquear a entrada de outros (como caminhões acima de um certo peso) ?



Para as perguntas 1, 2 e 3, fica claro que gerará tráfego na entrada, na saída e na via, respectivamente. Por fim, a resposta da pergunta 4 passa pela criação de regras prévias a partir da definição das políticas de filtro.

O mesmo pode ocorrer em um roteador, com a chegada de múltiplos *datagrams* em uma mesma *input port* sendo maior que sua capacidade de processá-los podendo gerar filas, atrasos e perda de dados (com o mesmo ocorrendo na saída). A quantidade limitada de *datagrams* suportada pelo *switch fabric* pode não suportar a alta demanda de fluxo de dados, causando o bloqueio de novos entrantes e, consequentemente, filas e atrasos.

Essas questões serão debatidas posteriormente em mais detalhes.


### Input Port

A Figura 02 mostra as execuções ocorridas na *input port*. Inicia-se com a ação de funções relativas a *physical layer* (em *Line Termination*), seguida de funções da *link layer* (em *Data link processing*), por fim o *lookup, fowarding* e *queuing* (em *lookup, fowarding, queuing*). 



| ![Image](imagens/input%20port.png)|
|:--------:|
|<b>Figura 02: Execuções na Input Port</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 314.</b>|


A função central da porta de entrada é o *lookup*, no qual o roteador procura (*look up*) na *forwarding table* qual porta o *datagram* recém chegado deve ser direcionado.
Apesar da proeminente importância do *lookup*, outras ações também devem ser tomadas, como a checagem do *version number*, *checksum* e *Time to Live*

Como pode-se supor, em uma arquitetura de banco de dados centralizado, os registros da *forwarding table* estariam disponíveis em um único local do roteador, com a operação de *look up* devendo fazer uma requisição de registro à esse banco de dados, algo que gera um possível gargalo, afinal, caso o módulo do banco de dados não conseguir suprir a demanda das requisições, deverão ocorrer filas e atrasos. 

Para evitar esse gargalo, cada linha da tabela é copiada para cada um dos *input ports* presentes no roteador, de forma que o acesso à tabela de transmissão (*forwarding table*) seja feita localmente (no *input port*), sem a necessidade de requisições.

No *look up*, o roteador identifica a saída (*link interface*) utilizando a regra de maior correspondência (*longest prefix matching rule*) de prefixo do IP *Address* de destino do *datagram*. Um exemplo dessas correspondências podem ser vistas na Tabela 01.
(O prefixo é formado por x primeiros dígitos do IP *Address* de destino referencia o endereço da subrede.)




Prefix | Link Interface
-----------|-------------
 11001000 00010111 00010  |      0
 11001000 00010111 00011000 |    1
 11001000 00010111 00011   |     2
 otherwise             |         3

<b>Tabela 1 Exemplo de forwarding table</b>



Por exemplo, o endereço de IP:

```
11001000 00010111 00011000 10101010
```

Tem o prefixo correspondendo ao *link interface* 1 e 2, com o *link* 1 sendo a maior correspondência e, portanto, sendo os dados direcionados ao mesmo.



### Switching


O *switching fabric* tem a função de conectar as *input ports* com as *output ports*. Há diferentes formas de implementá-lo, mas três delas, mostradas na Figura 03, destacam-se:




| ![Image](imagens/switching.png)|
|:--------:|
|<b>Figura 03: Tipos de Switching</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 317.</b>|


1. *Switching via memory*: o *input port* sinaliza, a partir de uma interrupção, para o processador do roteador a chegada de novos *datagrams*. Após a interrupçãos, os dados recém chegados são copiados para a memória, processados (determinando-se a interface apropriada), e por fim copiados para o *buffer* de saída. Isso é usualmente feito com processos com memória compartilhada.Como desvantagem pode-se citar, primeiro, a impossibilidade de transmissão de mais de um *datagram* ao mesmo tempo, pois somente uma operação de leitura e escrita na memória pode ser feita por vez. Segundo, com a largura de banda sendo B *datagrams* por segundo para memória (B *datagrams* podem ser escritos ou lidos em 1 segundo), significa dizer que a taxa de transferência está limitada em B/2 (pois será necessário fazer 1 operação de escrita e uma de leitura).
2. *Switching via bus*: a *input port* envia os *datagrams* diretamente para a sua respectiva *output port* através de um barramento compartilhado, sem a intervenção do processador do roteador. Isso é usualmente feito agregando-se um *header* com um rótulo informando sua respectiva interface. Ao ser transmitido, todas as interfaces recebem esses dados, porém somente aquela indicada pelo rótulo irá manter o mesmo, retirando o seu rótulo, processando-o, e transmitindo-o para fora do roteador. O primeiro fato incoveniente dessa arquitetura é que somente 1 pacote de dados podem pecorrer o barramento. Isso significa que se chegarem multiplos *datagrams* em diferentes *input ports*, todos menos 1 devem esperar o barramento tornar-se disponível. O segundo problema é que a velocidade do roteador estará limitada pela velocidade do barramento barramento. Esse tipo de arquitetura é indicado para LAN.
3. *Switching via an interconnection network*: também chamado de cruzamento de barras, esse tipo de switch, como mostrado na Figura 02, pode alternar cada cruzamento de barra (ou nó) entre aberto e fechado de forma independente. Dessa maneira, multiplos *datagrams* podem atravessar em paralelo o *switch fabric* ao mesmo (ou seja, *non-blocking* para uma *output port* específica). Por exemplo, para emitir um dado do *input* A para o *output* Y, basta fechar o cruzamento dos mesmos. Esse fechamento não impedirá que os dados do *input* B alcance a saída X, porém o *output* Y estará somente disponível para o *input* A (estando bloqueado para os outros *inputs*). 


### Output port


A Figura 04 mostra as 3 principais execuções do *output port*. Inicia-se com o enfileiramento (*Queuing*) dos *datagrams* recebidos. Em seguida, são performados ações necessárias relativas as camadas físicas e enlace (*Data link processing*). Por fim, os dados são enviados para fora do roteador (*Line termination*).



| ![Image](imagens/output%20port.png)|
|:--------:|
|<b>Figura 04: Output Port</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 319.</b>|

### Filas

As filas podem se formar na entrada e na saída do roteador.

As filas na entrada podem ser formadas pela diferença de velocidade entre o *input* e o *switching fabric* (como citado anteriormente, de forma análoga, na questão 1 do "E se" no tópico" Questões a a cerca do forwarding"). Se um *input* for capaz de lidar com uma taxa de *datagrams* mais alta do que o *switching fabric*, os mesmos ficarão acumulados na entrada até serem executados pelo *switching fabric*.

Um outro evento que têm como consequência a geração de filas é o chamado *head-of-the-line blocking* (HOL *blocking*). 
A Figura 05 mostra um exemplo de como o HOL *blocking* pode ocorrer. Os *datagrams* azuis-escuros estão destinados às saídas superiores, enquanto que os azuis-claros estão destinados às saídas centrais. Assim, um azul-escuro oriundo do *input* superior pode bloquear a passagem do *datagram* azul-escuro oriundo do *input* inferior, travando a fila e impedindo que o *datagram* azul-claro seja processado paralelamente, apesar do caminho até sua saída estar livre.



| ![Image](imagens/hol%20blocking.png)|
|:--------:|
|<b>Figura 05: Hol Blocking</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 321.</b>|

Na saída, as filas podem se formar quando multiplos *datagrams* dos *inputs* são direcionados para o mesmo *output*, como mostrado na Figura 06. Esse evento pode preencher o *buffer* de saída, ocasionando a "derrubada" de novos *datagrams*, política chamada de *drop-tail*, ou a remoção de um já enfileirado, para assim criar espaço para os dados recém chegados. Em alguns casos, pode ser vantajoso remover um pacote de dados antes que fila fique cheia, de forma a enviar um sinal de congestionamento para o emissor. Os algoritmos responsáveis por isso são chamados de *Active Queue Management* (AQM), ou gerenciador de filas ativo, com o *Random Early Detection* (RED) sendo um algoritmo dessa classe amplamente implementado.



| ![Image](imagens/output%20queue.png)|
|:--------:|
|<b>Figura 06: Fila na saída</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 322.</b>|

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

Na discurssão dos *buffers* fora deixado implícito a política *First-in-First-Out* (FIFO), ou primeiro a chegar, primeiro a sair (modelo mostrado na Figura 07), mas outras regras também podem ser utilizadas, como a classificação dos dados em diferentes filas a partir de sua prioridade (modelo mostrado na Figura 08).




| ![Image](imagens/FIFO.png)|
|:--------:|
|<b>Figura 07: Modelo FIFO</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 325.</b>|



| ![Image](imagens/priority%20queue.png)|
|:--------:|
|<b>Figura 08: Modelo classificação e priorização</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 326.</b>|

Uma forma generalizada de classificação e priorização dos dados que tem sido bastante implementada é o chamada de *Weighted Fair Queuing* (WFQ), na qual as filas de maior peso são tratadas primeiro, seguindo para as de menor peso, até reiniciar o ciclo, como mostrado na Figura 09.


| ![Image](imagens/WFQ.png)|
|:--------:|
|<b>Figura 09: Weighted Fair Queuing</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 326.</b>|

##### Neutralidade das redes



Como qualquer regra pode ser imposta para a classificação dos dados, os ISP's podem não ser neutros no oferecimento de seus serviços, algo que vem resultando leis que regulamentam o que os ISP's podem ou não fazer.

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


Dentro do padrão OpenFlow, os *headers* podem ser oriundos das camadas de enlace, rede e transporte, como mostrado na Figura 10. É importante perceber que nem todos os *headers* podem ser escolhidos para a ação de correspondência (*match*) (assim fora implementado como uma forma de equilibrar a funcionalidade e complexidade. É melhor fazer algo simples e bem, do que muita coisa e ruim).


| ![Image](imagens/Open%20Flow%20headers.png)|
|:--------:|
|<b>Figura 10: OpenFlow Headers</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 356.</b>|

As ações a serem tomadas são:

1. *Forwarding*: a transmissão dos dados.
2. *Dropping*: o "deixar de lado".
3. *Modify-field*: a modificação de algum *header*.


### Camada de rede: Control Plane

Na aula anterior, foi debatido a importância da *Fowarding Table* para o *Data Plane*. Os dados registrados nessa tabela são computados pela *Control Plane*, a qual tem como objetivo controlar a rota global que os *datagrams* precisarão pecorrer para sair de uma ponta à outra da rede (end-to-end). A *Control Plane* também configura e gerencia os componentes e serviços fornecidos pela camada de rede.

Uma rede pode ser vista como um grafo, no qual os vértices (ou nós) são os roteadores e as arestas são a conexão entre dois vértices, como pode ser visto na Figura 11. Dessa forma, os algorítmos de roteamento determinam o melhor caminho que um dado pode pecorrer para sair de um vértice da rede até outro vértice.

As caractarísticas de cada conexão (velocidade, tarifas financeiras, etc.) são contabilizadas (a partir de métricas estabelecidas pela instituição dona da rede), resultando no custo (ou peso) agregado à conexão. Como cada aresta apresenta características diferentes, serão agregados pesos diferentes ao uso de cada uma. Assim, os algorítmos de roteamento, como o OSPF e o BGP (conhecido como a "cola" da Internet), tem o objetivo de encontrar um caminho entre dois nós que apresente o menor custo de ser pecorrido (custo total do caminho).



| ![Image](imagens/grafo.png)|
|:--------:|
|<b>Figura 11: Grafo com pesos</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 381.</b>|



É importante perceber que o caminho de menor custo (*least-cost path*) é diferente do caminho mais curto (*shortest path*), pois o primeiro é caracterizado por aquele que apresenta o menor somátorio dos pesos das conexões inseridas no mesmo, enquanto que o segundo é determinado pela menor quantidade de nós que deve ser pecorrido.

Ambos os algoritmos citados (OSPF e BGP) utilizam a abordagem *per-router control* (mostrado na Figura 12), em que o algoritmo de roteamento é processado dentro de cada roteador, sendo necessário interações entre *routers* para a determinação das rotas. Outra possível abordagem é a *Logically centralized control* (mostrado na Figura 13), em que há uma centralização computação em um servidor e distribuição dos parâmetros determinados para os nós da rede, como adotado pelo SDN (*Software Defined Network*). 



| ![Image](imagens/per%20router%20control.png)|
|:--------:|
|<b>Figura 12: Per Router Control</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 378.</b>|



| ![Image](imagens/Logically%20centralized%20controller.png)|
|:--------:|
|<b>Figura 13: Logically Centralized Controller</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 379.</b>|



#### Classificação dos algoritmos

De forma abrangente, podemos classificar os algoritmos de roteamento em:

1. *Centralized routing algorithm*: os algoritmos dessa categoria, comumente referidos como *link-state* (LS) *algorithms*, computam o caminho a partir de um conhecimento completo a cerca da conectividade e custos de cada conexão da rede (tendo um conhecimento global da rede). Um exemplo é o algoritmo de Dijkstra.

2. *Decentralized routing algorithm*: a determinação do caminho de menor custo é feita de forma iterativa e distribuida em cada roteador. Como, inicialmente, cada roteador só têm conhecimento dos custos de seus próprios *links*, o cálculo do menor caminho necessitará da troca de informações entre os outros roteadores da rede. Isso ocorre de forma iterativa e gradual. Um exemplo é o *Distance-Vector Routing Algorithm* (DV), um algoritmo iterativo, assíncrono e distribuído, com cada nó mantendo um vetor de estimativas de custos de todos os outros nós da rede (com a atualização ocorrendo conforme mudanças na rede acontecem). Podemos citar alguns protocolos que utilizam o DV: Internet's RIP, BGP, ISO IDRP, Novel IPX e o original ARPAnet.


Uma segunda forma de classificação é:

1. *Static Routing Algorithms*: os roteadores mudam muito pouco no decorrer do tempo (frequentemente como resultado de uma intervenção humana).
2. *Dynamic routing algorithms*: as rotas mudam conforme a ocorre mudança na topologia da rede ou carga de tráfego (apesar dessa categoria apresentar maior responsividade em mudanças na rede, também estão mais susetíveis a problemas como *loops* e oscilações).

Uma terceira forma:

*Load-Sensitive algorithm*: os custos dos *links* variam dinamicamente conforme o nível de congestionamento.
*Load-Insensitive algotihm*: os custos dos *links* não refletem explicitamente o nível de congestionamento.


##### LS vs DV

Alguns pontos são importantes para a comparação entre os *link-state algorithms* e o *Distance-Vector Routing Algorithm*:

1. *Message complexity*: o LS requer que cada nó saiba os custos de cada conexão da rede, com uma mudança em um custo devendo ser enviada para todos os nós. Já no DV, o envio do novo custo somente ocorre quando o mesmo impacta no caminho de menor custo dos nós anexados ao *link* relativo à alteração (DV melhor que LS).

2. *Speed of convergence*: LS converge mais rápido do que o DV (LS melhor que DV).

3. *Robustness*: no DV, um caminho de menor custo calculado incorretamente por um nó será publicado para todos os nós da rede, diferentemente do LS, no qual os caminhos são cálculados em cada nó, provendo assim um certo nível de robustez (LS melhor que DV).



#### Intra-Autonomous Sistems Routing: OSPF

Um sistema autônomo (Autonomous Sistems, AS) consiste de um conjunto de roteadores que estão sob um mesmo controle administrativo. Isso torna possível a escalabilidade e a autonomia administrativa do sistema. Os algoritmos de roteamento que rodam em um AS são chamados de *intra-autonomous system routing protocol*. Um exemplo de algoritmo é o *Open Shortest Path First*, um protocolo LS no qual as especificações do protocolo de roteamento está disponível publicamente (a parte *Open* do nome). No OSPF, cada roteador monta um mapa topológico completo de todo o AS, e então executa o algoritmo de Dijkstra para a determinação da árvore de caminhos de menor custo para todas as subredes, com os custos das conexões sendo configurados pelo administrador da rede.

Observe que is roteadores que conectam-se com outros de diferentes ASs são chamados de *gateway routers* (eles estão nos limites da AS). Já aqueles que estão no interior da AS, são chamados de *internal routers*.

Alguns pontos importantes de mencionar:

1. *Security*: as trocas de informações entre os roteadores OSPF podem ser autenticadas por meio da geração de um *hash* MD5 (utilizando-se uma chave secreta, que está contida no roteador e não é compartilhada) por ambos (emissor e receptor da mensagem). Em seguida o *hash* do emissor é comparado com o do receptor. O emissor pode ser considerado autêntico caso os *hashs* gerados sejam iguais. Caso contrário, a mensagem dese ser ignorada.

2. *Multiple same-cost path*: O tráfego de dados poderá ser distribuído em todas as rotas que contenham o custo mínimo.

3. *Support for hierarchy within a single AS*: o sistema pode ser configurado hierarquicamente em áreas.


#### Inter-Autonomous Sistems Routing: BGP

Por necessitar a coordenação de múltiplos ASs, a comunicação entre ASs deve ocorrer utilizando-se o mesmo protocolo. O protocolo usado é o *Border Gateway Protocol* (BGP), e é conhecido também por ser a "cola" da Internet (por unir os diferentes ASs).

O roteamento pelo BGP ocorre entre subredes e não para um endereço específico da rede. Dessa forma, a *forwarding table* do roteador toma a forma de `(x, I)`, onde `x` é o prefixo e o `I` é a interface do roteador. 

O BGP provê para os roteadores meios para:

1. Obter o prefixo de AS vizinhos: com a publicação da existência de cada rede para o resto da Internet.
2. Determinar a melhor rota para cada prefixo: no qual a melhor rota é baseada nas políticas determinadas pelo administrador da rede e na acessibilidade da informação.

As conexões BGP entram em duas categorias (graficamente representado na Figura 14):

1. iBGP (*internal* BGP): conexão BGP internas aos ASs
2. eBGP (*external* BGP): conexão externa aos ASs (entre ASs)


| ![Image](imagens/eBGP%20e%20iBGP.png)|
|:--------:|
|<b>Figura 14: eBGP e iBGP</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 402.</b>|

A publicação no BGP ocorre de forma bem direta. A Figura 15 mostra a adição de uma subrede (`x`) em uma rede com 3 ASs. Primeiro o AS3 envia uma BGP *message* (`AS3 x`) para o AS2 dizendo que a subrede `x` existe e é acessível através dele. Em seguida, o AS2 avisa para o AS1 a existência de `x` e que o mesmo é acessível através do caminho AS2-AS3 (`AS2 AS3 x`). 



| ![Image](imagens/SAs%20com%20a%20adicao%20de%20x.png)|
|:--------:|
|<b>Figura 15: ASs com adição de x</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 401.</b>|

O BGP *message* é composto pelo prefixo e outros múltiplos atributos, como o *AS-PATH*, que explicita a lista de ASs na qual a mensagem publicação de existência da nova rede precorreu (como mostrado no exemplo anterior), e *NEXT-HOP*, que é o endereço da interface do roteador que inicia a o *AS-PATH*.


#### Hot Potato


Os ASs operam com a bordagem *Hot Potato*, o qual os roteadores objetivam transmitir os dados para fora do AS o mais rápido possivel. Para tal, o roteador com os dados disparará para o endereço do *NEXT-HOP* que tiver o menor custo de conexão, sem se preocupar com o resto do trajeto desses dados. Assim, apesar de localmente eficiente, a rota global escolhida pode não ser a mais rápida. A Figura 16 mostra duas possibilidades de *NEXT-HOP*. A rota escolhida será aquela que apresentar o menor custo de conexão relacionado ao *NEXT-HOP*. A Figura 17 mostra os passos para a adição de um destino externo ao AS na *forwarding table*



| ![Image](imagens/rota%20com%20menor%20next-hop.png)|
|:--------:|
|<b>Figura 16: Duas possibilidades de NEXT-HOP</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 403.</b>|



| ![Image](imagens/adicao%20destino.png)|
|:--------:|
|<b>Figura 17: Passos para a adição de um destino externo na forwarding table</b> 
<b>Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 404.</b>|



##### Seleção da rota.

Na prática, a rota selecionada utiliza outros parâmetros além do *Hot Potato*:

1. Política de decisão da preferência local de transmissão: essa política é escolhida pelo administrador da rede
2. Para os restantes, será escolhido o que tiver o menor *AS-PATH*.
3. Para os restantes, o *Hot Potato* entra em ação, escolhendo a rota com o menor custo de transmissão para o *NEXT-HOP*.
4. Para os restantes, é utilizado os indentificadores BGP para a seleção da rota.


##### Política de preferência local e Protocolos Intra vs Inter 

O AS é dito como sistema autônomo pois o mesmo apresenta uma idenpendência administrativa. Isso é algo que pode gerar alguns problemas de confiança, com descisões como não permitir que dados originário de um AS passe através de outro AS específico. E, de forma similar, um AS pode ter interesse em querer controlar o tráfego de dados entre outros ASs. Um reflexo no cenário atual é a desconfiança dos americanos sob empresas chinesas, que, comumentemente, recebe interferência do governo chinês, um inimigo declarado dos EUA (assim, deve-se ser evitado que dados críticos do governo americanos sejam transmitidos através de empresas chinesas).

Outra questão gira em torno do problema de uma sudrede de um cliente fazer parte da rota dos dados de outros clientes. Um cliente deve ser origem ou destino das mensagens e não um meio.


A política é uma questão tão forte na comunicação inter-ASs, que ela é um dos maiores motivos para que os protocolos inter-ASs sejam diferentes dos intra-ASs, tanto que até a qualidade das rotas (externas) utilizadas é uma preocupação secundária. Por fim, a questão da escala não é tão forte no intra-ASs quanto é em inter-ASs. Assim, os 3 motivos para que os protocolos sejam diferentes são:

1. Política: intra-ASs é secundário, inter-ASs é primário.
2. Escala: intra-ASs é secundário, inter-ASs é primário.
3. Performace: intra-ASs é primário, inter-ASs é secundário (impactado pelas políticas).

