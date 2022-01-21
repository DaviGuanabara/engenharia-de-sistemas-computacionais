## Rotatória e roteador: Fowarding function

Imagine uma via de veículos com pedágio. Suas entradas direcionam todos os carros de diferentes origens para um mesmo pedágio. Durante o pedágio, os carros aguardam pacientemente em uma fila a execução de uma série de operações, como o pagamento da taxa de uso da via. Após essas operações, os carros atravessam a via de transporte encaminhando-se até as suas respectivas saídas.

O roteador funciona de forma análoga a essa via imaginada. Os *datagrams*, análogos aos carros, após entrarem no roteador (pelos *inputs*), são enfileirados e ficam no aguardo de serem processados. Após uma série de operações, os mesmos são direcionados para as suas respectivas sáidas (*outputs*). 


..................


Várias outras vias anexam-se a ela  é delimitada por um pedágio, e o seu "corpo" conecta-se com  


Um roteador é form
Um roteador é formado por 3 componentes, as portas de entrada, o comutador, e as portas de saída. De forma análoga, um roteador pode ser visto como uma via de carros, com uma rotatória no centro.


1. Porta de entrada (*input port*) (vias de entradas de carros): análogo ás vias de entrada de carros, é responsável por executar funções relacionadas as camadas físicas e enlace, com a função mais crucial sendo a de *lookup* (a determinação da porta de saída)
