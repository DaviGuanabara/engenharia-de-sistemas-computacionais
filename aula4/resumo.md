Como dito na aula anterior, um processo é uma entidade ativa a qual se refere à um programa em execução. Porém, a CPU não recebe essa 
entidade na sua integralidade para processamento, e sim uma parte da mesma chamada de Thread. Nessa está contido os elementos necessários para
a mudança de contexto da CPU (*CPU Context Switch*), como os valores dos registradores utilizados, a memória Stack e o PC (Program Counter). 
É importante notar que os sistemas operacionais evoluíram, com os mais antigos limitando-se à *single-threading* (uma *thread* por processo), e os mais novos rompendo essa
limitação, passando à *multi-threading* (múltiplas *threads* por processo).

quebrando a limitação de uma única thread por processo.
Nos sistemas mais antigos, cada processador tinha, exclusivamente, uma única thread, sendo assim um sistema *single-thread*. 

![Figura 1](single-multi-threaded.png)
*Figura 1: Single and Multi-Threaded Example*
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 160.*
