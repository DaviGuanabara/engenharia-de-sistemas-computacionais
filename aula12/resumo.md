### Continuação: *Transport Layer*

Como citado na aula anterior, o protocolo *stop and wait*, no qual o envio do próximo segmento (também é chamado de pacote por uma questão histórica) ocorre somente após o recebimento da resposta do segmento anterior, é uma forma ineficiente para a transferência de dados confiáveis (*reliable data transfer*, rdt) pois, após a transmissão de um segmento, os recursos disponíveis para a conexão ficarão osciosos até a chegada da resposta do segmento enviado. Dessa maneira, objetivando o aumento da eficiência do mesmo, fora propostos duas soluções: *Go-Back-N* (bgn); e *selective repeat* 

#### *GO-BACK-N*

A ideia do protocolo gbn é basear-se em um subconjunto de N elementos da fila de transmissão (um dos motivos para a imposição de um tamanho limite é o controle de fluxo). Os elementos dessa fila são compostos por espaços que podem ser preenchidos por segmentos oriundo das camadas superiores. Esse subconjunto, chamado de janela, contém os espaços preenchidos por segmentos enviados mas sem confirmação (*acknowledged*) e espaços ainda não preenchidos. Ao receber uma resposta, o espaço relacionado ao respectivo segmentos sai da janela, um novo elemento da fila de transmissão é adicionado, gerando o efeito de deslizar da janela para a direita na fila de transmissão, devido a esse efeito o gbn é chamado de *sliding-window protocol*. 

Caso todos os espaços disponibilizados pela janela estejam preenchidos, novos dados não poderão ser aceitos, retornando ás camadas superiores, sendo esse retorno uma indicação de indisponibilidade.

Na metade superior da Animação 01 pode ser identificado os parâmetros: `base`, que identifica o valor inicial incluido; `nextseqnum`, referente ao próximo elemento a ser enviado; e `send window size`, tamanho da janela (valor do N supracitado). A metade inferior mostra o registro dos eventos ocorridos durante o protocolo. 

Animação 01: Animação Go-Back-N\
![Alt Text](imagens/animação%20gbn.gif)

Disponível em: https://media.pearsoncmg.com/aw/ecs_kurose_compnetwork_7/cw/content/interactiveanimations/go-back-n-protocol/index.html


Como o envio dos segmentos é feito em ordem, é esperado que os respectivos ACK's sejam recebidos em ordem (*cumulative acknowledgment*). Caso o *server* receba um segmento corrompido ou fora de ordem, o mesmo é descartado e um ACK referente ao último segmento íntegro ordenado é disparado. O ACK duplicado recebido é descartado. Da perspectiva do *client*, a não recepção ACK correspondete ao segmento enviado, pode resultar em dois casos. Primeiro, se a recepção do ACK 'x + 1' ocorrer, porém a do 'x' não, o gbn considerará que o segmento 'x' foi recebido corretamente pelo *server* e o seu ACK fora perdido durante a transmissão, marcando, portanto, o segmento 'x' como enviado corretamente. O segundo caso é a respeito da não recepção de respostas dentro de um período predeterminado (*timeout*), algo que resulta na retransmissão dos segmentos relativos.

Exemplo 1:

1. *cliente* dispara 5 segmentos (1, 2, 3, 4, 5)
2. *server* recebe o segmento número 2 corrompido
3. *server* dispara os seguintes ACK's: 1, 1, 1, 1, 1
4. *cliente* recebe 5 ACK's referentes ao segmento 1, indicando a necessidade de retransmissão dos segmentos 2, 3, 4 e 5.

Exemplo 2:

1. *client* dispara 5 segmentos (1, 2, 3, 4, 5)
2. *server* recebe todos os segmentos corretamente
3. *server* dispara os seguintes ACK's: 1, 2, 3, 4, 5
4. *client* somente o ACK número 5, tendo, o resto, sido perdido durante a transmissão.
5. *client* qualifica todos os 5 segmentos como tendo sido recebidos corretamente pelo *server*



#### Selective Repeat (SR)

É importante perceber, como mostrado no Exemplo 1, que um único elemento corrompido pode causar a retransmissão de uma série de segmentos, tornando, assim, o gbn ineficiente para esses casos. O aumento dessa ineficiência é diretamente proporcional ao número de erros provocados pelo canal de transmissão.


O protocolo *Selective Repeat* (SR), como o próprio nome já induz, tem o objetivo de diminuir o número de retransmissões desnecessárias. Para tal, utiliza um subconjunto de N elementos da fila de transmissão, chamada de janela, com cada elemento sendo um espaço que pode ser preenchido por um segmento e marcado como não usável, usável, enviado e confirmado, algo análogo ao gbn. Porém, diferencia-se pelo seu comportamento. 

1. um elemento só é marcado como confirmado quando o mesmo receber seu respectivo ACK. 
2. a janela desloca-se somente após o recebimento do ACK relativo ao elemento na posição `base`, localizado no início da janela. 
3. o *server* enviará um ACK para cada segmento mesmo se ele estiver fora de ordem. 
4. o *client* terá uma janela própria (do mesmo tamanho da janela do *client*), organizando-a, também, com 4 marcadores: esperado; fora de ordem; aceitado; não usável. 
5. um segmento recebido fora de ordem não será descartado e sim armazenado em uma memória temporária (*buffer*).

Esse comportamento gera uma possível desincronização da posição das janelas do *cliente* e do *server*, pois o *server* pode receber adequadamente um segmento mas o seu ACK relativo ter sido corrompido ou perdido durante a transmissão. O pior caso de desincronização ocorre quando todos os ACK's enviados tenham sido perdidos e, consequentemente, o *server* estará adiantado em `N` elementos. Assim, caso o *server* receba um segmento com o número da sequência entre entre os intervalos:

1. [`base`, `base` + `N` - 1]: armazenar em memória temporária.
2. [`base`-N, `base` - 1]: reenviar o ACK.
3. Fora dos anteriores: ignorar.

A desincronização pode causar a recepção de um segmento já recebido, gerando um dilema para o receptor: o segmento recebido é novo ou é uma retransmissão ?

Essa possível desincronização pode causar um dilema para o receptor dos dados, pois, como mostrado na 
A Figura 01 mostra o dilema ocasionado pela possível dessincronização:  os dados recebidos são derivados de uma retransmissão, como mostrado em (a), ou de um novo segmento (b)? 

Figura 01: Dilema do Selective Repeat\
![Image](imagens/Dilema%20do%20Selective%20Repeat.png)
Imagem retirada de: Computer Networking a top-down approach. 8th ed. Pearson, página 225.


Colocações importantes:

1. O número de sequência é um valor finito determinado pelo número de bits disponibilizados para tal.
2. O *buffer* do receptor deve poder armazenar 2 janelas, ou seja, o seu tamanho deve ser, no mínimo, o dobro do `window size` (`N`).
3. O `N` é determinado durante o *handshaking*, limitado pelo tamanho do *buffer* do receptor (como citado em 2).



#### Revisão de TCP

1. Ponto-a-ponto
2. Transmissão de confiança, com fluxo de dados ordenado
3. *Full Duplex*: fluxo de dados bi-direcional na mesma conxeção
4. *cumulative* ACK's
5. *Pipelining*: controle de fluxo e congestionamento
6. *Connection-oriented*
7. Fluxo controlado: o emissor não irá sobrecarregar o receptor
