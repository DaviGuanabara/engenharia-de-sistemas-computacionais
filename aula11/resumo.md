
### *Transport-Layer*

O objetivo da camada de transporte (*transport layer*), executada nos dispositivos localizados nas pontas da comunicação, é extender as funcionalidades da *network layer* com a preparação e envio dos dados enviados pelos diferentes *sockets*, técnica chamada de *multiplexação*, e o direcionamento do pacote recebido para o *socket* competente, procedimento chamado de demultiplexação. Dessa maneira, do ponto de vista da aplicação, a *transport layer* provê uma comunicação lógica, como se os processos estivessem interconectados diretamente, algo similar ocorre na *network layer*. Porém os diferentes protocolos existentes nessa camada podem fornecer serviços adicionais, como confiabilidade da transferência dos dados e controle de congestionamento, ambos presentes no protocolo TCP (*Transmission Control Protocol*, ou Protocolo de Controle de Transmissão) e não encontrados no UDP (*User Datagram Protocol*).

A multiplexação e demutiplexação são possíveis devido à estrutura de dados *4-tuple*, no qual está contido os endereços de IP da origem e do destino, e os identificadores únicos de 16 *bits*, chamado de porta (*port*), dos *sockets* de origem e destino. 

Diferentemente do UDP, no qual a demultiplexação ocorre com o encaminhamento dos dados diretamente para a porta de destino descrita no *header* do *segment*  (ou seja, o *socket* UDP é completamente identificável com *2-tuple*, ou estrutura de dados que contém o IP e *port* de destino), o TCP necessita da *4-tuple* completa para a identificação do respectivo socket. Essa abordagem simplifica a arquitetura *client-server*, pois o servidor pode utilizar de um única porta (como uma *well-known port number*, número de porta que varia de 0 até 1023) para a execução do *handshaking*, como a 80 no caso do HTTP, enquanto que entrega dinamismo à criação, ocorrendo no *request*, e finalização dos *sockets*, algo que sucede o encerramento do canal de comunicação.
Na Figura 01 pode ser observado como os dados contidos na *4-tuple* podem variar conforme o protocolo escolhido.

É importante perceber que um processo pode conter multiplos *sockets*, com cada um associando-se à uma *thread*. Assim, em servidores HTTP atuais, por exemplo, cada *request* recebido gera um novo conjunto *thread* e *socket* e o fim da conexão encerra esse conjunto. Portanto, o período de vida do *thread* e *socket* pode ser longo, durando toda a comunicação no modo persistente, ou curto, com a conexão encerrando-se logo após o envio do *response* no modo não persistente.
Requisições frequentes no modo não persistente podem impactar no desempenho do sistema.


Figura 01: Multiplexação e Demultiplexação \
![image](imagens/Multiplexação%20e%20Demultiplexação.png)


