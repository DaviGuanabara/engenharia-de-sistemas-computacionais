### Continuação: *Transport Layer*

Como citado na aula anterior, o protocolo de enviar o próximo pacote somente após o recebimento da resposta do pacote anterior é uma solução ineficiente para a transferência de dados confiáveis (*reliable data transfer*, rdt). Para aumentar a eficiência do processo, fora propostos dois protocolos alternativos: *Go-Back-N* (bgn); e *selective repeat* 

#### *GO-BACK-N*

A ideia do protocolo gbn é basear-se em um subconjunto de N elementos da fila de transmissão. Os elementos dessa fila são compostos por espaços que podem ser preenchidos por pacotes oriundo das camadas superiores. Esse subconjunto, chamado de janela, contém os espaços preenchidos por pacotes enviados mas sem confirmação (*acknowledged*) e espaços ainda não preenchidos. Ao receber uma resposta, o espaço relacionado ao respectivo pacote sai da janela, um novo elemento da fila de transmissão é adicionado, criando-se o efeito de deslizar da janela para a direita da fila de transmissão, e por isso esse protocolo é chamado de *sliding-window protocol*.

A ideia do protocolo gbn é de transmitir um subconjunto de até N pacotes, nos quais os N foram a chamada janela de envio. Conforme 
