## Retomando aos processos

Nas últimas, foram debatidos diversos aspectos dos processos dos SO (Sistemas Operacionais). Esses aspectos orbitam os 3 pontos enfatizado a seguir:



1. Identificar os componentes de um processo.
2. Criação e Término de processos
3. Comunicação entre processos: *Shared Memory* x *Message Passing*.



### Componentes de um processo.

Enquanto que um programa é uma entidade passiva localizada na memória secundária, um processo é instância ativa de um programa, e está localizado na memória principal em quatro sessões básicas (Como mostrado na Figura 1): *Text*, código executável; *Data*, variáveis globais; *Heap*, memória dinamicamente alocada; *Stack*, armazenamento de dados temporários.

*Figura 1: Processo na memória principal*

![Figura 1](C%20code%20in%20memory.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 106.*

Do ponto de vista do Sistema Operacional, cada processo é representado pelo *Process Control Block* (PCB), mostrado na Figura 2, o qual contém vários pedaços de informação necessários para iniciar ou reiniciar um processo específico, como *Process State*, *Program counter*, *CPU registers* e *CPU-scheduling information*.


*Figura 2: PCB: Representação do processo no SO*

![Figura 2](Process%20Block%20Control%20(PCB).png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 109.*


De forma geral, os processos podem assumir 5 estados (Como mostrado na Figura 3): criado, pronto, rodando, esperando e terminado.


*Figura 3: Estados de um processo*

![Figura 3](Diagram%20of%20Process%20State.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 109.*



No Linux, todos os PCB's encontram-se em uma lista duplamente encadeada (com o próximo e o anterior), com o sistema operacional armazenando um ponteiro para a estrutura que está atualmente em execução pela CPU. Mostrado na Figura 4.

*Figura 4: Representação da fila encadeada*

![Figura 4](Representação%20da%20fila%20encadeada.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 111.*

Conforme o estado do processo muda, o PCB também muda de fila, alternando entre a fila encadeada de "pronto" (pronto para ser executado) e espera (espera de ocorrência de evento), mostradas na Figura 5. 


*Figura 5: Filas de espera e pronto*

![Figura 5](Filas%20de%20espera%20e%20pronto.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 112.*


O responsável por depositar e recolher os PCB's dessas filas é o *process Scheduling*, o qual foi representado no diagrama da Figura 6.


*Figura 6: Diagrama de agendamento de processos*

![Figura 6](Diagrama%20Agendamento%20de%20Processos.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 113.*


O *CPU Scheduling*, por sua vez, tem como papel recolher um processo da fila *ready* para um núcleo de processamento. Para ser alocado na CPU, é necessário salvar o estado do processo anteriormente alocado, bem como os elementos necessários para continuar processando-o posteriormente, e carregar o novo contexto de execução. No decorrer dessa tarefa, chamada de *Context Switch*, a CPU fica ociosa, como mostra a Figura 7.


*Figura 7: Diagrama da troca de contexto*

![Figura 7](Diagrama%20Contex%20Switch.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 113.*


### Criação e Término de um processo.


Cada processo tem uma identificação única chamada de `pid` (*process identifier*). A partir dele pode ser criado um processo filho. Conscutivas criações formam uma árvore de processos, mostrada na Figura 8.

*Figura 8: Árvore de processos típico de sistema Linux*

![Figura 8](Árvore%20de%20processos%20em%20um%20sistema%20linux.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 116.*

O ciclo de criação e término de um processo, mostrado na Figura 9, passa pelas funções de chamada de sistema na linguagem `C`: `fork()`, para criar um novo processo; `wait()`, esperar o processo filho acabar; `execve()`, troca a imagem de execução (troca o programa); `exit()`, encerrar o processo.

*Figura 9: Criação e término de um processo*

![Figura 9](Process%20Creation.png)

*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 119.*

Alguns sistemas não permitem que um processo "viva" sem que o "pai" exista. Então, se o "pai" for encerrado, todos os "filhos" serão encerrados. Da mesma forma que os filhos dos filhos. Fenômeno chamado de *cascading termination*

Ao chamar a função `exit()`, um processo filho tem seus recursos deslocados pelo sistema operacional. Porém ele continua na tabela de processos até o processo pai executar a função `wait()`, pois nessa tabela está contido o estado de saída do mesmo. Durante essa transição, de seus recursos terem sido deslocados mas ainda estar na tabela de processos, o processo é chamado de `Zumbie`, o que, normalmente, ocorre para todos os processos (ao serem terminados) por um curto período de tempo.
Se o pai não chamar a função `wait()` e terminar sua execução, os processos filhos passarão a ser chamados de processos `órfãos`. O Sistema Operacional Linux utiliza de outros processos para herdar os `órfãos` e libera-los da tabela de processos chamando a função `wait()`.



### Comunicação entre processos: *Shared Memory* x *Message Passing*.

Um processo pode ter sido criado com o objetivo de executar um algoritmo que independe de outros processos (esse processo é chamado de independente). Ou, pode ter sido criado para cooperar com outros processos (esse processo é chamado de cooperativo). Para que a cooperação ocorra, é necessário criar um canal de comunicação (um *Interprocess Communication - IPC* ), de tal maneira que os dados possam ser transmitidos de uma para outra. Há dois métodos para tal: o *Shared Memory* (compartilhamento de memória); e o *Message Passing* (sistema de passagem de mensagem).

Há várias razões para ter um processo cooperativo.
1. Troca de informações: Compartilhamento de informações entre os interessados.
2. Aceleração computacional: Dividir uma tarefa em várias pequenas partes e, assim, processá-los ao mesmo tempo.
3. Modularidade: Com as funções do sistema dividido em diferentes processos ou threads.


#### Shared Memory

*Shared Memory* é o método que torna uma faixa de endereços de memória acessível à diferentes processos. Por quesões de segurança, o sistema operacional (de forma padrão) não permite que os processos tenham acesso aos espaços de memória dos outros. Para que esse método ocorra, é fundamental que todos os processos participantes desse método concordem em compartilhar a sua faixa de memória. Após executado, o sistema operacional não mais intervem, deixando para os processos a responsabilidade de garantir que não ocorra o problema chamado *Race Conditions* (ou condição de disputa).

O *Race Conditions* decorre de um problema de *concurrency* (simultaneidade) entre os diferentes processos, como a escrita em um mesmo endereço de memória transcorrendo por mais de um processo ao mesmo tempo.

Uma solução para garantir a comunicação entre os processos é utilizando a arquitetura *producer-consumer*, no qual o processo *producer* produz e armazena o conteúdo em um buffer, que será consumido pelo *consumer* posteriormente. Esse *buffer* pode ser *unbounded buffer* (não tem limites de tamanho) ou *bounded buffer* (com o tamanho limitado).


#### Message Passing


A idéia por de trás do método *message passing* é a de passar a responsabilidade da distribuição e validade da comunicação interprocessual para o sistema operacional, promovendo uma abstração de uso facilitado (em comparação ao *shared memory*), pois descarta a necessidade da criação explícita de um código de gerenciamento de acesso e manupulação dos dados compartilhados. Esse mecanismo de comunicação cria um elo de ligação (*link*) entre dois processos, que demanda ao menos duas operações: *send(message)*; e *receive(message)*.

A implementação desse método passa por 3 pontos:

1. Comunicação direta e indireta. 
2. Síncrona ou assíncrona.
3. *Buffer* automático ou explícito.

##### Comunicação Direta
Na comunicação direta, o processo deve nomear explicitamente o nome do receptor ou emissor:
1. *send(P, message)*: Enviar menssagem para o processo P
2. *receive(Q, message)*: Receber mensagem do processo Q

##### Comunicação indireta
Na comunicação indireta, as mensagens passam através de uma abstração de caixa de correios (*mailbox* ou *ports*) compartilhada, na qual as mensagens podem ser postadas e removidas. Cada *mailbox* tem uma identificação única (no POSIX, cada *mailbox* é identificado por um inteiro). Nessa implementação, o processo deve nomear de qual e para qual caixa de correios deve enviar ou receber a mensagem:

1. *send(A, message)*: Enviar uma mensagem para o *mailbox* A.
2. *receive(A, message)*: Receber uma mensagem do *mailbox* A.


Um possível *Race Condition* ocorre quando o processo P1 envia uma mensagem, e o P2 e P3 estão chamando a função `receive()`. Para evitar que isso aconteça, existem 3 possibilidades:

1. Permitir que somente 2 processos estejam associados à uma caixa de correios.
2. Permitir que somente 1 processo por vez execute a função `receive()`
3. Permitir que o sistema operacional escolha qual processo deve receber (P1 ou P2, mas não ambos).


##### Sincronia
Para cada um dos dois primitivos (`send()` e `receive()`), o sistema de transmissão de mensagens pode assumir 2 estados, o síncrono e o assíncrono:

1. Síncrono (*Blocking* ou bloqueante): 
1.1. O emissor (P1) é travado até o receptor (P2) receber a mensagem.
1.2. O receptor (P1) é travado até uma mensagem ficar disponível.

2. Assíncrono (*Nonblocking* ou não-bloqueante)
2.1. O emissor (P1) continua a executar suas instruções após a emissão da mensagem.
2.2. O receptor (P2) pode receber uma mensagem inválida (NULL), assim continuar sua execução mesmo sem ter recebido uma mensagem válida.


##### *Buffer*

Todas as mensagens enviadas devem passar por um sistema de fila que, basicamente, tem 3 formas distintas (que apenas variam sua capacidade):

1. Capacidade 0 (*Zero capacity* ou sem *buffer*): O emissor deve ser síncrono com o receptor, caso contrário a mensagem será perdida.
2. Capacidade limitada (*Bounded capacity*): O emissor pode ser assíncrono caso haja capacidade de armazenamento de mensagens no *buffer*. E, caso a fila esteja cheia, deve ser bloqueado até haver espaço disponível.
3. Capacidade ilimitada (*Unbounded capacity*): O emissor nunca é bloqueado, pois a fila não tem limites de recepção de mensagens.





### Complemento

Essa sessão faz alusão à outros conteúdos também importantes relacionados com a aula, mas que dará ao leitor o papel de buscar mais informações sobre os mesmos conforme sua necessidade e interesse.



#### Threads - criação em `C`


Nas aulas anteriores foram discutidos as vantagens das threads em comparação aos processos. Abaixo está um código comentado em `C` para criação de threads.

```C
#include <stdio.h>
#include <pthread.h> //POSIX thread library, or pthread.


int sum;
void *runner(void *param);

int main(int argc, char *argv[])
{
        pthread_t tid;
        pthread_attr_t attr;
        
        pthread_attr_init(&attr); //Atributo necessário para a criação da thread
        
        /*
         * pthread_create
         * tid vai retornar o 'id' da thread. 
         * runner é a função que será executada
         * argv[1] é o argumento no qual será passado para a função runner
         *
         */
        pthread_create(&tid, &attr, runner, argv[1]); 
        pthread_join(tid, NULL); //Espera uma thread Terminar
        
        return 0;
}

void *runner(void *param){
        
        int upper = atoi(param);
        int sum = 0;
        
        for(int i = 0; i < upper; i++){
               sum += 1;
        }
        
        pthread_exit(0);
}
```

#### Pesquisar sobre

1. Teoria de filas
2. *pipes* (mecanismo *IPC - InterProcess Communication*) - Página 139 (Capítulo 3.7) de Silberschatz, A. Operating System Concepts, 10th.





