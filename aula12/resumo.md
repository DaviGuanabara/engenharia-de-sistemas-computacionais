### Continuação: *Transport Layer*

Como citado na aula anterior, o protocolo *stop and wait*, no qual o envio do próximo segmento (também é chamado de pacote por uma questão histórica) ocorre somente após o recebimento da resposta do segmento anterior, é uma forma ineficiente para a transferência de dados confiáveis (*reliable data transfer*, rdt) pois, após a transmissão de um segmento, os recursos disponíveis para a conexão ficarão osciosos até a chegada da resposta do segmento enviado. Dessa maneira, objetivando o aumento da eficiência do mesmo, fora propostos duas soluções: *Go-Back-N* (bgn); e *selective repeat* 

#### *GO-BACK-N*

A ideia do protocolo gbn é basear-se em um subconjunto de N elementos da fila de transmissão (um dos motivos para a imposição de um tamanho limite é o controle de fluxo). Os elementos dessa fila são compostos por espaços que podem ser preenchidos por segmentos oriundo das camadas superiores. Esse subconjunto, chamado de janela, contém os espaços preenchidos por segmentos enviados mas sem confirmação (*acknowledged*) e espaços ainda não preenchidos. Ao receber uma resposta, o espaço relacionado ao respectivo segmentos sai da janela, um novo elemento da fila de transmissão é adicionado, gerando o efeito de deslizar da janela para a direita na fila de transmissão, devido a esse efeito o gbn é chamado de *sliding-window protocol*.

Na metade superior da Animação 01 pode ser identificado os parâmetros: `base`, que identifica o valor inicial incluido; `nextseqnum`, referente ao próximo elemento a ser enviado; e `send window size`, tamanho da janela (valor do N supracitado). A metade inferior mostra o registro dos eventos ocorridos durante o protocolo. 

Animação 01: Animação Go-Back-N\
![Alt Text](imagens/animação%20gbn.gif)

Disponível em: https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/go-back-n-protocol/index.html


Caso a janela já esteja, os dados oriundos das camadas superiores não são aceitos e retornam aos mesmos, indicando-lhes que a janela não suporta novos dados.
*timeout event*


Caso algum segmento seja perdido durante o envio, o receptor enviará, para cada novo segmento recebido fora de ordem, a resposta referente ao último segmento em ordem, como mostrado na Animação 02.

Se um segmento é recebido corretamente e em ordem, o receptor envia um ACK confirmando sua recepção. Para todos os outros casos, os segmentos recebidos são descartados e o receptor reenvia o ACK referente ao último segmento em ordem.

Se o receptor receber o ACK referente ao segmento n + 1 e não tiver recebido o ACK referente ao n, 
Por o envio dos ACK's 

Uma característica do protocolo gbn é o chamado *cumulative acknowledgment*, 

Como o envio dos segmentos é feito em ordem, é esperado que os respectivos ACK's sejam recebidos em ordem. Caso o *client* receba um segmento corrompido ou fora de ordem, o mesmo é descartado e um ACK referente ao último segmento íntegro ordenado é disparado. Da perspectiva do *server*, a não recepção ACK correspondete ao segmento enviado, pode resultar em dois casos. Primeiro, se a recepção do ACK 'x + 1' ocorrer, porém a do 'x' não, o gbn considerará que o segmento 'x' foi recebido corretamente com o seu ACK perdido durante a transmissão, marcando, portanto, o segmento 'x' como enviado corretamente. O segundo caso é a respeito da não recepção de respostas dentro de um período predeterminado (*timeout*), algo que resulta na retransmissão dos segmentos relativos.
