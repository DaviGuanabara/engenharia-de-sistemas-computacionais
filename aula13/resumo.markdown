## IP, o protocolo da network layer

O objetivo da *network layer* (camada de rede) é transferir dados de um *host* emissor para o *host* receptor (como não há garantias, esse serviço é conhecido como *best-effort service*). Para tal, é necessário determinar a rota (*route* ou *path*) global que os dados devem pecorrer para alcançar o seu destino, chamado de *routing*, bem como especificar o caminho local com o direcionamento dos dados recém chegados no roteador para a sua saída apropriada, chamado de *fowarding*. 

Dessa forma, a *network layer* pode ser decomposta em duas partes, *control plane* e *data plane*, nas quais estão contidos o *routing* e o *fowarding*, respectivamente.

É importante perceber que apesar dessas duas funcionalidades serem requisitos para que a *network layer*, é possível encontrá-las em dispositivos separados, algo possibilitado pelo SDN (Software-Defined Networks), que aloca o *control plane* em um servidor, algo que torna os roteadores especialistas em *fowarding*.

A *fowarding table*, responsável por determinar, a partir do *header* dos dados recebidos pelo roteador à sua saída apropriada, é o elemento chave para a interação entre essas partes, pois seus registros são gerados pelo *control plane* e utilizados por *data plane*.



