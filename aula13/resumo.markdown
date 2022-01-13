## IP, o protocolo da Network Layer

O objetivo da *network layer* (camada de rede) é transferir dados de um *host* emissor para o *host* receptor (como não há garantias, esse serviço é conhecido como *best-effort service*). Para tal, é necessário determinar a rota (*route* ou *path*) global que os dados devem pecorrer para alcançar o seu destino, chamado de *routing*, bem como especificar o caminho local com o direcionamento dos dados recém chegados no roteador para a sua saída apropriada, chamado de *fowarding*. 

Dessa forma, a *network layer* pode ser decomposta em duas partes, *control plane* e *data plane*, nas quais estão contidos o *routing* e o *fowarding*, como mostrado na Figura 01.

Figura 01: Control plane e data plane\
![Image](imagens/Control%20plane%20and%20data%20plane.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 307.

É importante perceber que apesar dessas duas funcionalidades serem requisitos para que a *network layer*, é possível encontrá-las em dispositivos separados, algo possibilitado pelo SDN (Software-Defined Networks), que aloca o *control plane* em um servidor, algo que torna os roteadores especialistas em *fowarding*.

A *fowarding table*, responsável por determinar, a partir do *header* dos dados recebidos pelo roteador, a saída apropriada, é o elemento chave para a interação entre essas partes (*control plane* e *data plane*), pois seus registros são gerados pelo *control plane* e utilizados pelo *data plane*.




### Protocolo IP

O IP (*Internet Protocol*) é o protocolo utilizado na camada de rede. Atualmente está largamente em uso as versões 4 e 6, as quais serão discutidas a seguir. A estrutura de dados resultante da *network layer* é o *datagram*, no qual o seu arranjo varia conforme a versão utilizada.

Citando o livro-texto desse curso:

"Nevertheless, the datagram plays a central role in the internet - every networking student and professional needs to see it, absorb it, and master it." (Computer Networking a top-down approach. 8th ed. Pearson, página 331)

entender o *datagram* é de suma importância para a aprendizagem de redes de computadores.


#### IPv4 Datagram 

O *datagram*, como mostrado na Figura 02, segue o seguinte formato:

1. Version Number: especifica a versão utilizada (no caso, 4)
2. Header length: a versão 4 do IP tem um tamanho de *header* variável, causado pelo campo *options*, o que torna necessário a manutenção dessa informação dentro do *datagram* (o campo *options* não é normalmente utilizado, portanto o tamanho típico do *header* IPv4 *datagram* é de 20 *bytes*)
3. Type of Service (TOS): exemplos são *real time*, *non-real-time*.
4. Datagram length: o tamanho do *datagram* é calculado pela soma dos tamanhos do header dos dados carregados (*payload*). Composto por 16 bits e medido em *bytes*, o tamanho teórico máximo é de 65535 bytes (raramente ultrapassa os 1500 bytes).
5. Identifier, flags e fragmentation offset: utilizados para fragmentar um *datagram* muito grande em pedaços pequenos que serão enviados independentemente e remontados no destino.
6. time-to-live (ttl): decrementa em 1 toda vez que o *datagram* é processado por um roteador, sendo descartado ao atingir 0.
7. Protocol: indica o protocolo da camada de transporte. contém o valor de 6 (00110) para TCP e 17 (10001) para UDP.
8. Header Checksum: Objetivando detectar erros, esse campo é computado tratando cada 2 bytes do **header** como um número e somando-os utilizando a aritmética complementar de 1. Esse cáculo é somente feito para o **header**, evitando possíveis redundâncias com as camadas inferiores.
9. Source e destination IP addresses.
10. Options: permite uma extensão do *header*. Não foi incluido no *datagram* do IPv6 por dificultar a determinação do início do *payload* (necessitando do campo *header length*) e a variação do tempo requerido para processamento (pois alguns *datagrams* podem requisitar ou não o processamento do campo *options*).
11. Data (payload): contém o segmento da camada de transporte.


Figura 02: IPv4 Datagram\
![Image](imagens/IPv4%20Datagram.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 331.


#### Endereçamento

A fronteira entre o *host* e a conexão física é chamada de interface. Em um roteador é possível identificar multiplas interfaces. Como cada interface está associado um endereço de IP, um roteador, portanto, pode estar associado a múltiplos endereços de IP.
O endereço de IP é formado por 4 bytes (32 bits), escritos na notação *dotted-decimal*, no qual cada byte é escrito em decimal e separado por um ponto.

O endereçamento segue a estratégia conhecida como *Classless Interdomain Routing* (CIDR), no qual o endereço de IP (formado por 32 bits) é separado em prefixo (os `x` primeiros bits) e identificador de host (os restantes `32 - x` bits). O número de bits (`x`) que formam o prefixo é determinado pela máscara de rede (*mask subnet*), como mostrado a seguir:




```
Endereço de IP: a.b.c.d
Endereço de IP com máscara de rede: a.b.c.d/x
Notação: 193.32.216.9/24 (11000001 00100000 11011000 00001001)
Prefixo (Subrede): 193.32.216 (11000001 00100000 11011000)
Identificador de Host: 9 (00001001)
```

A máscara de subrede (*network mask*) distingue o endereço referente à subrede ao do *host*. A subrede pode ser entendida como uma ilha de rede isolada, com as interfaces compondo as bordas dessa rede, como mostrado na Figura 03.

Figura 03: Subrede\
![Image](imagens/subnet.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 336.

##### Obter um endereço de IP


A obtenção de um endereço de IP ocorre de forma automática com o protocolo DHCP (*Dynamic Host Configuration Protocol*), chamado também de protocolo *plug-and-play* ou *zeroconf* (*zero configuration*), o qual gera um endereço de IP temporário diferente toda vez que o *host* conecta-se nessa rede.
O DHCP é um protocolo baseado na arquitetura *client-server*, e o seu processo é feito em quatro passos (mostrado na Figura 04):

1. Server Discovery: o *client* dispara *datagram* contendo um *discovery message* para o destino 255.255.255.255 (esse endereço de IP indica ao roteador que a mensagem deva ser entregue para todos as interfaces da subrede), com a origem em 0.0.0.0.
2. Server Offer: O Servidor DHCP, após receber a discovery message*, responde com a *offer message* para 255.255.255.255 (ou seja, disparando para todos os dispositivos da subrede). A *offer message* contém: uma propósta de *IP address*; *network mask*; e o *IP address lease time*, referente à validade do IP. É importante perceber que na mesma rede pode haver múltiplos servidores DHCP e, portanto, múltiplas *offer message* podem ser disparadas durante essa etapa.
3. Request: o *client* escolhe uma *server offer* e ecoa os seus parâmetros com a *request message*.
4. ACK: o servidor selecionado confirma a seleção do endereço enviando uma *ACK message*.

Figura 04: Processo DHCP\
![Image](imagens/DHCP%20process.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 343.


##### NAT


1. Como um servidor DHCP local geraria um IP único diferente de um outro servidor geograficamente distante ?
2. Os diferentes servidores DHCP devem estar sincronizados ou faixas específicas de IP devem ser pré-determinadas ?
3. Eu já encontrei redes domésticas com a mesma faixa de endereço de IP, como isso é possível ?

Essas e outras perguntas vem à tona quando é imaginado como funcionaria uma interação global de subredes.
A solução passa pelo uso do protocolo NAT (*Network Address Translation*), mostrado na Figura 05.


Figura 05: NAT\
![Image](imagens/NAT.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 345.


Um roteador com o protocolo NAT ativo é visto como um dispositivo único (com o IP único) para o resto do mundo, escondendo, assim, os detalhes das configurações de uma rede doméstica para as redes externas. 

É interessante notar que o roteador obtém o endereço de IP via o servidor DHCP oriundo do ISP (*Internet Service Provider*). E por sua vez, oferece um servidor DHCP para a sua subrede.

O segredo para o funcionamento do NAT está no *NAT translation table*.

Quando uma requisição é disparada por um *host* para um *server* fora da rede, o roteador converte o endereço de IP de origem para o seu, e gera uma nova porta. Dessa forma, a *NAT translation table* é populada, no lado WAN (rede do ISP), com endereço de IP do roteador e porta gerada, e no lado LAN (rede doméstica), endereço de IP do *host* e porta da *Thread*.

Por exemplo:

1. *Host* dispara um *datagram* com origem 10.0.0.1 e porta 3345 para o *server* 128.119.40.186 porta 80.
2. O roteador gera uma nova porta e substitui os parâmetros de origem para essa porta gerada e para o seu endereço de IP (que por sua vez foi gerado pelo ISP). E registra essa conversão no *NAT translation table*. 
3. O *server* recebe o *datagram* com os parâmetros de origem do roteador, e o responde.
4. O roteador converte os parâmetros de destino utilizando a tabela NAT, e, por fim, direciona o *datagram* recebido ao *host*.

Esse registro é mantido até o fim da conexão. Como o tamanho do campo porta é de 16 bits, o protocolo NAT suporta mais de 60 mil conexões com somente um único *IP address*.

Um dos problemas causados por esses protocolos (DHCP e NAT) é referente aos *Home Servers*, pois como um servidor espera por uma requisição de um *client*, como esse *client* pode saber qual é o atual endereço de IP do servidor ? Como funcionaria a arquitetura P2P ?
Soluções para esse problema incluem *NAT transversal tools* [RFC 5389, RFC 5128], algo que não será debatido nesse texto.  

#### IPv6 datagram

Há uma série de mudanças introduzidas com o IPv6, mostrado na Figura 06:

Figura 06: IPV6 Datagram\
![Image](imagens/IPv6%20datagram.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 349.

1. Capacidade de endereçamento expandido: de 32 bits para 128 bits
2. *header* de tamanho fixo: o *header* foi fixado em 40 *bytes*, permitindo um processamento mais rápido pelo roteador
3. fluxo e seu rótulo: capacidade de rotular os *datagrams* de um fluxo específico, o qual cada rótulo sinaliza uma requisição específica, como *real-time service* ou *non-default quality*


Os campos do *datagram* do IPv6 são:

1. Version: versão do IP (no caso, 6).
2. Traffic class: equivalente ao TOS do IPv4, é usado para dar prioridade à algum *datagram*
3. *flow label*: campo de 20 *bits* usado para rotular um fluxo de *datagrams*
4. *Payload Length*: campo de 16 *bits* que indica o tamanho do *payload*
5. *Next Header*: identifica o protocolo para qual o *datagram* (ou o *payload*) será entregue
6. *Hop limit*: equivalente ao TTL do IPv4, diminui em 1 toda vez que o *datagram* é transmitido por um roteador e descartado quando chega em 0.
7. Endereço de origem e destino: no formato de 128 *bits*
8. *Data*: também chamado de *payload*, é o conteudo encapsulado pelo protocolo.

Foram removidos:

1. Fragmentação e remontagem: IPv6 não permite a fragmentação e a remontagem do datagram (essa operação era feita com o objetivo de reduzir o tamanho do *datagram* grande demais para ser transmitido). Assim caso um *datagram* seja muito grande, o roteador descarta esse *datagram* e retorna uma mensagem de erro, de forma que o emissor deverá reenviar o *datagram*. A remoção dessa funcionalidade gera, como resultado, uma redução no tempo de processamento do roteador.
2. *Header checksum*: Considerado suficientemente redundante e enfatizando a velocidade de processamento, esse campo não se mostrou necessário para os desenvolvedores do IPv6.
3. Options: a remoção do *options* possibilitou a fixação do tamanho do *header* em 40 *byts*, algo que além de causar um encolhimento no tempo de processamento, também deixa explícito o início do *payload* (afinal, o *payload* sempre estará 40 bytes após o início do *datagram*). 


#### Transição de IPv4 para IPv6

Há um problema inerente na atualização dos sistemas distribuidos: a adoção de uma nova tecnologia por todos os elementos da rede
Um sistema com o IPv6 pode ser projeto para ser retrocompatível com o IPv4, mas um sistema com IPv4 não é capaz de lidar com o IPv6.

Como, então, atualizar todos os incontáveis dispositivos já integrados na rede ?

1. Transição abrupta: substituir todos os elementos de uma vez só, marcando um dia x para inutilizar os dispositivos compatíveis com IPv4.
2. Transição suave: substituir os elementos de forma gradual.


A abordagem da transição suave foi o caminho escolhido. Para tal, fora adotado a prática do *tunneling* (algo que torna os dispositivos IPv6 compatível com o IPv4). O *tunnel* encapsula o *datagram* do IPv6 integralmente, tornando-o o *payload* do IPv4, resultando em um *datagram* com o *header* do IPv4.

Figura 07: Tunneling\
![Image](imagens/Tunneling.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 352.


## Link Layer

Após a *Network Layer* determinar qual o caminho de comunicação (chamado de *link* ou enlace) o *datagram* deve pecorrer, como o *WiFi* ou o *Ethernet*, entra em cena o *Link Layer* (camada de enlace), responsável por encapsular o *datagram* e transmitir o resultado (o *frame*) através do *link*. Os dispositivos que executam a camada de enlace são chamados de nós (*nodes*). 

### Serviços

A camada de enlace provê os seguintes serviços:

1. *Framing*: constituição do *frame* a partir do encapsulamento do *datagram*.
2. *Link access*: Um protocolo de 
