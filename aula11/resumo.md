
### *Transport-Layer* (Aula dia 10/11)

O objetivo da camada de transporte (*transport layer*), executada nos dispositivos localizados nas pontas da comunicação, é extender as funcionalidades da *network layer* com a preparação e envio dos dados enviados pelos diferentes *sockets*, técnica chamada de *multiplexação*, e o direcionamento do pacote recebido para o *socket* competente, procedimento chamado de demultiplexação. Dessa maneira, do ponto de vista da aplicação, a *transport layer* provê uma comunicação lógica, como se os processos estivessem interconectados diretamente, algo similar ocorre na *network layer*. Porém os diferentes protocolos existentes nessa camada podem fornecer serviços adicionais, como confiabilidade da transferência dos dados e controle de congestionamento, ambos presentes no protocolo TCP (*Transmission Control Protocol*, ou Protocolo de Controle de Transmissão) e não encontrados no UDP (*User Datagram Protocol*).


A multiplexação e demutiplexação, ambas demonstradas graficamente na Figura 01, são possíveis devido à estrutura de dados *4-tuple*, no qual está contido os endereços de IP da origem e do destino, e os identificadores únicos de 16 *bits*, chamado de porta (*port*), dos *sockets* de origem e destino.

Diferentemente do UDP, no qual a demultiplexação ocorre com o encaminhamento dos dados diretamente para a porta de destino, descrita no *header* do *segment*, (ou seja, o *socket* UDP é completamente identificável com *2-tuple*, ou estrutura de dados que contém o IP e *port* de destino), o TCP necessita da *4-tuple* completa para a identificação do respectivo socket. Essa abordagem simplifica a arquitetura *client-server*, pois o servidor pode utilizar de um única porta (como uma *well-known port number*, número de porta que varia de 0 até 1023) para a execução do *handshaking*, como a 80 no caso do HTTP, enquanto que entrega dinamismo à criação, ocorrendo no *request*, e finalização dos *sockets*, algo que sucede o encerramento do canal de comunicação.
Na Figura 01 pode ser observado como os dados contidos na *4-tuple* podem variar conforme o protocolo escolhido.

É importante perceber que um processo pode conter multiplos *sockets*, com cada um associando-se à uma *thread*. Assim, em servidores HTTP atuais, por exemplo, cada *request* recebido gera um novo conjunto *thread* e *socket* e o fim da conexão encerra esse conjunto. Portanto, o período de vida do *thread* e *socket* pode ser longo, durando toda a comunicação no modo persistente, ou curto, com a conexão encerrando-se logo após o envio do *response* no modo não persistente.
Requisições frequentes no modo não persistente podem impactar no desempenho do sistema.


Figura 01: Multiplexação e Demultiplexação \
![image](imagens/Multiplexação%20e%20Demultiplexação.png)



### *Reliable Data Transfer Protocol*

Para entender melhor sobre como funciona uma transferência de dados confiável (*Reliable Data Transfer Protocol*, RDT), será adotada uma máquina computacional abstrada com um número finito de estados, no qual assume somente um único estado por vez, chamada de Máquina de Estados Finitos (*Finite-State Machine*, FSM).

A cada nível de RDT será adicionado uma nova camada de complexidade, até chegar no modelo mais próximo do real.


#### RDT 1.0


No primeiro caso, consideramos que as camadas mais baixas são confiáveis. Ou seja não há perda ou alteração de dados e nem alteração na ordem de envio.
Assim:

Emissor:

1. rdt_send(data)
2. packet=make_pkt(data)
3. udt_send(packet)


Receptor:


4. rdt_rcv(packet)
5. extract(packet,data)
6. deliver_data(data)



#### RDT 2.0

Em RDT 2.0, vamos considerar que, durante a transmissão, algum bit pode ter sido corrompido. Assim, é necessário utilizar o protocolo ARQ (*Automatic Repeat reQuest*), o qual é baseado em três pontos: detecção de erro, permitindo o receptor reconhecer a ocorrência de erro; o *feedback* do receptor, com o parecer positivo (equivalente à "entendi!") chamado de ACK ou negativo (equivalente à "pode repetir?"), denominado de NAK, (em princípio, esse retorno basta ser de 1 bit de tamanho, sendo 0 para negativo e 1 para positivo); e retrasmissão, com o emissor reenviando os pacotes caso tenha recebido uma negativa (NAK) do receptor.


Emissor:

Envia os dados

1. rdt_send(data)
2. sndpkt = make_pkt(data,checksum)
3. udt_send(sndpkt)


Espera por uma resposta \
(reemissão em caso de NAK)
7. rdt_rcv(rcvpkt) && isNAK(rcvpkt)
8. udt_send(sndpkt)


(Encerra estado em caso de ACK) \
9. rdt_rcv(rcvpkt) && isACK(rcvpkt)


Receptor:
(Caso dados corrompidos)

4. rdt_rcv(rcvpkt) && corrupt(rcvpkt)

(Resposta) \
5. sndpkt=make_pkt(NAK)
6. udt_send(sndpkt)

(Caso dados não-corrompidos)

4. rdt_rcv(rcvpkt) && notcorrupt(rcvpkt)
5. extract(rcvpkt,data)
6. deliver_data(data)

(Resposta) \
7. sndpkt = make_pkt(ACK)
8. udt_send(sndpkt)


É importante perceber alguns detalhes desse protocolo. Primeiro, esse protocolo é conhecido por *stop-and-wait* pois o emissor não pode receber nenhum comando do seu operador enquanto estiver esperando uma resposta do receptor. Isso só ocorre quando a máquina estiver em um estado apropriado. Segundo, não foi considerada a possibilidade do ACK e NAK estarem corrompidos.

Para o segundo caso, há duas variações do RDT 2.0 no qual é implementado um protocolo análogo ao incremento feito de RDT 1.0 para 2.0, o RDT 2.1 e 2.2.

#### RDT 3.0

Por fim, vamos considerar que poderá haver pacotes, ou suas respostas ACK, perdidos durante a transmissão.
A solução é o emissor esperar tempo suficiente no qual garanta que os dados enviados ou respondidos foram perdidos. Assim, caso o tempo seja ultrapassado, o emissor deve reenviar os pacotes.

Como pode-se observar, não há como determinar um tempo "garantidor", pois um atraso particularmente alto experimentado pelos pacotes pode ser suficiente para ultrapassar o tempo estimado. 

A determinação do tempo sofre de uma dicotomia, pois há vantagens e desvantagens tanto com o aumento como com a diminuição do mesmo. Quanto menor for o tempo, maior a chance de ocorrer falsas perdas (ocasionando duplicações na emissão) por consequência de atrasos na rede. Porém, incrementos nesse tempo podem impactar na velocidade da resposta do emissor, pois o mesmo poderá ficar longos períodos de inatividade aguardando uma resposta (perdida durante a transmissão). Assim, a melhora na efetividade do protocolo passa pela derterminação de um tempo de espera ótimo (provavelmente desenvolvido para o atraso mais frequênte).

A Figura 02 mostra um exemplo do funcionamento do protocolo RDT 3.0 (também conhecido por *alternating-bit protocol*, por causa da alternância no número da sequência dos pacotes).

Figura 02: Protocolo RDT 3.0 \
![image](imagens/rdt%203.0.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 212.

#### Performace

O fundamento dos protocolos mencionados é enviar 1 pacote e esperar por sua resposta. Uma forma de melhorar a performace é utilizar um pardão de enviar multiplos pacotes antes de entrar no estado de espera da resposta de cada um, como mostrado na Figura 03.


Figura 03: *Stop-and-wait vs pipelined*\
![image](imagens/stop%20and%20wait%20vs%20pipelined.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 213.

