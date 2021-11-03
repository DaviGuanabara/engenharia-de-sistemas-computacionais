
### Camada de Aplicação: a razão de existir das redes

  
A camada de aplicação (primeira camada) é considerada como a mais importante, pois ela é quem gera a utilidade da rede. Assim, a infraestrutura, bem como os protocolos, foram criados para servi-la, e, portanto, sem a aplicação, não existiria as demais camadas. Essa utilidade vem dos benefícios produzidos pela interação de dois processos executados em computadadores diferentes e, possivelmente, sistemas operacionais diferentes. Para tal, há duas principais arquiteturas, *client-server* e *peer-to-peer* (P2P).

#### HTTP
A arquitetura *client-server* é empregue na aplicação *Web*, o qual usufrui do protocolo *Hypertext Transfer Protocol* (HTTP). Essa arquitetura funciona no sistema *always on*, no qual o servidor, ou processo que espera para ser contactado no início da interação, deve sempre ficar ativo, e o seu endereço na rede fixado. Os clientes não interagem diretamente entre si. O protocolo de transporte usado é o TCP. 

A escalabilidade dessa arquitetura (a capacidade de suportar o aumento do número de clientes) está limitada pela capacidade da infraestrutura já instalada. Assim, o aumento no número de *requests* (derivado do aumento no número de *clients*) deve ser precedido por um aumento na competência do servidor de lidar com os mesmos. Por essa razão, os servidores são comumentes hospedados em *data centers*, algo que facilita a escalabilidade, por alocar, dinamicamente, centenas de dispositivos para o processo *server* (o servidor se torna uma entidade processada por mútiplos dispositivos).

O HTTP define a estrutura das mensagens e a forma que as mesmas são trocadas. Essa troca é baseada no padrão *response-request* (a mensagem do *client* é o *request*, e a reposta do *server* o *response*). O *client* (uma navegador como o *Chrome*, por exemplo), definido por ser o processo que inicia a comunicação, necessita do endereço do *server*, chamado de URL, que é composta por dois componentes, o *hostname* ( *http://www.google.com/*) e o *pathname* (*/search?q=alguma+coisa*):

```
URL: http://www.google.com/search?q=alguma+coisa
```    

A mensagem HTTP *request* segue o seguinte padrão:

*Request Line*         : GET /somedir/page.html HTTP/1.1 \r\n  (= Método + URL + HTTP *version* + *carriage return and line feed*)
*Header lines* (line 1): HOST: www.google.com.br \r\n          (*host* no qual hospeda o objeto requisitado)
*Header lines* (line 2): Connection: Mozilla/5.0 \r\n          (navegador *Firefox*)
*Header lines* (line 3): Accept-language: fr \r\n              (prefere a versão em língua francesa)
*Header lines* (line x): ... \r\n                              (outras configurações)
*Entity body* (line x): \r\n                                   (pode ficar vazio, como no método *GET*, ou preenchido, como no método *POST*)


A mensagem HTTP *response* segue o seguinte padrão:

*Status Line*          : HTTP/1.1 200 OK \r\n                                  (Tudo ocorreu bem)
*Header lines* (line 1): Connection: close \r\n                                (*server* irá encerrar a conexão)
*Header lines* (line 2): Date: Tue, 18 Aug 2015 15:44:04 GMT \r\n              (data em que o *response* foi enviado)
*Header lines* (line 3): Server: Apache/2.2.3 \r\n                             (nome do servidor)
*Header lines* (line 4): Last-Modified: Tue, 18 Aud 2015 15:11:03 GMT          (data da última modificação)
*Header lines* (line 5): Content-Length: 6821 \r\n                             (número de bytes do objeto enviado - O mesmo está em *Entity body*)
*Header lines* (line 6): Content-Type: text/html \r\n                          (indica o formato do objeto, no caso uma página html)
*Entity body*          : ... \r\n                                              (dados do objeto)




#### Torrent

A segunda arquitetura citada anteriormente, a P2P, basea-se na intercolaboração intermitente dos dispositivos na rede, ou *peers*, de forma que esses podem ser *client* e *server* simultaneamente (É importante perceber que, em uma sessão de comunicação com um par de dispositivos, o *client* é aquele que inicia o contato e o *server* aquele que espera para ser contactado, assim, apesar de cada *peer* assumir ambas as formas simultaneamente nessa rede, elas tem um único papel para cada sessão). Essa arquitetura é auto-escalável (*self-escalability*), pois cada *peer*, apesar de adicionar novos *requests*, adicionam também capacidade de serviço na rede.

A popularidade desse protocolo vem do uso da tecnologia *Torrent*. Desenvolvida pela empresa *BitTorrent*, essa tecnologia descentraliza o fornecimento de arquivos, de forma que os *peers*, conforme fazem o *download*, também proporcionam os dados já baixados de volta para a rede. Assim, a capacidade de transmissão não está limitada pelo *seed* (primeiro fornecedor de um conteúdo), crescendo, e, consequentemente, tornando arquivos populares mais acessíveis, conforme aumenta a participação dos usuários (sendo, consequentemente, barato, por não necessitar a contratação de uma infraestrutura para o fornecimento de arquivos).

Essa arquitetura enfrenta diversos desafios, como: segurança, performace e confiabilidade.


### Sockets

Os *sockets* são uma estrutura de dados em *kernel space* (encontrados dentro do sistema operacional, ou seja, seu acesso é através de uma chamada de sistema) que fornece (analogamente a uma porta), uma interface (API, *Application Programming Interface*) para a interação da camada de aplicação com a camada de transporte. Ele se manifesta através de um FD (*file descriptor*, um arquivo), fornecendo um *pipeline* para o processo (em *user-space*).
Para ser executado, o *socket* necessita do endereço do *host* (*internet protocol address* ou *IP address*) e do identificador que especifica o processo receptor no *host* (chamado de *Port Number*). Por padrão, o *web server* utiliza a porta 80, e o *mail server* (com o protocolo SMTP) é identificado pela porta 25. 


### Serviços de transporte

Como citado anteriormente, a camada de transporte fornece uma série de serviços para a camada superior. Esses serviços podem ser analisados sob a ótica de: confiabilidade (*reliable data transfer*), taxa de transferêria (*throughput*), atraso (*delay* ou *timing*) e segurança (*security*). Nesse sentido, a *internet* prover dois protocolos de transporte, o UDP e o TCP, os quais disponibiliza serviços com características diferentes, de maneira que:

TCP:

1. *Handshaking procedure*: com uma preparação anterior ao envio da mensagem (confiabilidade).
2. *Full duplex connection*: com os processos podendo receber e enviar dados.
3. *Reliable data transfer service*: sendo os dados enviados sem erros e na ordem correta (confiabilidade).
4. *Transport Layer Secutiry: uma melhora no TCP que fornece criptografia (*encryption*), integridade de dados (*data integrity*) e *end-point authentication* (segurança).
5. Garantia de entrega da mensagem.


UDP (um *lightweight transport protocol*):

1. *No handshaking procedure*: Não se sabe se o receptor está preparado para receber a mensagem.
2. *Unreliable data transfer*: podendo ser recebidos dados corrompidos e fora de ordem.
3. Não há garantia de entrega da mensagem.
4. Rápida.

É importante perceber que os serviços de *timing* e *throughput*, apesar de serem necessários para algumas aplicações, como *internet telephony* (Skype), não são fornecidos pelos protocolos de *internet* atuais.



### Conteúdos Adicionais

1. *Socket programming*: página 152 do livro *Computer Networking a top-down approach 8th ed.*
2. *Dual Stack*: IPv4 e IPv6
3. NAT
4. WireShark: www.wireshark.org
5. ip addr shwo
6. ip config /on
7. telnet 127.0.0.1 3000
8. SS - apt | grep 3000
9. SS - apt | grep telnet
10. nc - V - U 127.0.0.1 1200
11. ncat
12. CDN (*Content Distribution Networks*) 
13. DASH (*Dynamic Adaptive Streaming over HTTP*).
14. DNS (*domain name system*): The Internet’s Directory Service
15. telnet serverName 25
16. SMTP (*Simple Mail Transfer Protocol*)
17. *Cookies* (*User interaction*)
