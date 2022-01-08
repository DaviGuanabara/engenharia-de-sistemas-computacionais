## IP, o protocolo da network layer

O objetivo da *network layer* (camada de rede) é transferir dados de um *host* emissor para o *host* receptor (como não há garantias, esse serviço é conhecido como *best-effort service*). Para tal, é necessário determinar a rota (*route* ou *path*) global que os dados devem pecorrer para alcançar o seu destino, chamado de *routing*, bem como especificar o caminho local com o direcionamento dos dados recém chegados no roteador para a sua saída apropriada, chamado de *fowarding*. 

Dessa forma, a *network layer* pode ser decomposta em duas partes, *control plane* e *data plane*, nas quais estão contidos o *routing* e o *fowarding*, respectivamente.

É importante perceber que apesar dessas duas funcionalidades serem requisitos para que a *network layer*, é possível encontrá-las em dispositivos separados, algo possibilitado pelo SDN (Software-Defined Networks), que aloca o *control plane* em um servidor, algo que torna os roteadores especialistas em *fowarding*.

A *fowarding table*, responsável por determinar, a partir do *header* dos dados recebidos pelo roteador, a saída apropriada, é o elemento chave para a interação entre essas partes (*control plane* e *data plane*), pois seus registros são gerados pelo *control plane* e utilizados pelo *data plane*.

### Protocolo IP

O IP (*Internet Protocol*) é o protocolo utilizado na camada de rede. atualmente está largamente em uso as versões 4 e 6, as quais serão discutidas a seguir. A estrutura de dados resultante da *network layer* é o *datagram*, no qual o seu arranjo varia conforme a versão utilizada.

E, citando o livro-texto desse curso:

"Nevertheless, the datagram plays a central role in the internet - every networking student and professional needs to see it, absorb it, and master it." (Computer Networking a top-down approach. 8th ed. Pearson, página 331)

entender o *datagram* é de suma importância para a aprendizagem de redes de computadores.


#### IPv4 Datagram 

O *datagram* segue o seguinte formato:

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



#### Endereçamento

a fronteira entre o *host* e a conexão física é chamada de interface.

