## Retomando aos processos

Nas últimas, foram debatidos diversos aspectos dos processos dos SO (Sistemas Operacionais). Esses aspectos orbitam os 6 pontos enfatizado a seguir:



1. Identificar os componentes de um processo.
2. Criação e Término de processos
3. Comunicação entre processos: *Shared Memory* x *Message Passing*.
4. Exemplos de comunicação com pipes e *POSIX Shared Memory*.
5. Arquitetura Produtor-Consumidor
6. Projetar Módulos do kernel que interajam com o Linux


Para o capítulo atual, foi proposto uma breve introdução aos elementos já debatidos anteriormente, com a adição de novos elementos ainda não citados.

### Componentes de um processo.

Enquanto que um programa é uma entidade passiva localizada na memória secundária, um processo é instância ativa de um programa, e está localizado na memória principal em quatro sessões básicas (Como mostrado na Figura 1): *Text*, código executável; *Data*, variáveis globais; *Heap*, memória dinamicamente alocada; *Stack*, armazenamento de dados temporários.

*Figura 1: Processo na memória principal*
![Figura 1](Layout%20of%20a%20process%20in%20memory.png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 106.*

Do ponto de vista do Sistema Operacional, cada processo é representado pelo *Process Control Block* (PCB), mostrado na Figura 2, o qual contém vários pedaços de informação necessários para iniciar ou reiniciar um processo específico, como *Process State*, *Program counter*, *CPU registers* e *CPU-scheduling information*.


*Figura 2: PCB: Representação do processo no SO*
![Figura 2](Process%20Block%20Control%20(PCB).png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 109.*


De forma geral, os processos podem assumir 5 estados (Como mostrado na Figura 3): criado, pronto, rodando, esperando e terminado.


*Figura 3: Estados de um processo*
![Figura 3](Diagram%20of%20Process%20State.png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 109.*



No Linux, todos os PCB's encontram-se em uma lista duplamente encadeada (com o próximo e o anterior), de forma que o sistema operacional armazena um ponteiro para a estrutura que está atualmente em execução pela CPU. Mostrado na Figura 4.

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


O *CPU Scheduling*, por sua vez, tem como papel recolher unm processo da fila *ready* para um núcleo de processamento. Para ser alocado na CPU, é necessário salvar o estado do processo anteriormente alocado, bem como os elementos necessários para continuar processando-o posteriormente, e carregar o novo contexto de execução. No decorrer dessa tarefa, chamada de *Context Switch*, a CPU fica ociosa, como mostra a Figura 7.


*Figura 7: Diagrama da troca de contexto*
![Figura 7](Diagrama%20Contex%20Switch.png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 113.*


### Criação e Término de um processo.


Cada processo tem uma identificação única chamada de pid. A partir dele pode ser criado um processo filho. Conscutivas criações formam uma árvore de processos, mostrada na Figura 8.

*Figura 8: Árvore de processos típico de sistema Linux*
![Figura 8](Árvore%20de%20processos%20em%20um%20sistema%20linux.png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 116.*

O ciclo de criação e término de um processo, mostrado na Figura 9, passa pelas funções de chamada de sistema na linguagem `C`: `fork()`, para criar um novo processo; `wait()`, esperar o processo filho acabar; `execve()`, troca a imagem de execução (troca o programa); `exit()`, encerrar o processo.

*Figura 9: Criação e término de um processo*
![Figura 9](Process%20Creation.png)
*Imagem retirada de: Silberschatz, A. Operating System Concepts, 10th, página 119.*

Alguns sistemas não permitem que um processo "viva" sem que o "pai" exista. Então, se o "pai" for encerrado, todos os "filhos" serão encerrados. Da mesma forma que os filhos dos filhos. Fenômeno chamado de *cascading termination*

Ao chamar a função `exit()`, um processo filho tem seus recursos deslocados pelo sistema operacional. Porém ele continua na tabela de processos até o processo pai executar a função `wait()`, pois nessa tabela está contido o estado de saída do mesmo. Durante essa transição, de seus recursos terem sido deslocados mas ainda estar na tabela de processos, o processo é chamado de `Zumbie`, o que, normalmente, ocorre para todos os processos em um curto período de tempo.
Se o pai não chamar a função `wait()` e terminar sua execução, os processos filhos passarão a ser chamados de processos `òrfãos`. O Sistema Operacional Linux utiliza de outros processos para herdar os `òrfãos` e libera-los da tabela de processos chamando a função `wait()`.



### Comunicação entre processos: *Shared Memory* x *Message Passing*.

Um processo pode ter sido criado com o objetivo de executar um algoritmo que independe de outros processos (processo chamado de independente). Ou, pode ter sido criado para cooperar com outros processos (processo chamado de cooperativo). Para que a cooperação ocorra, é necessário criar um canal de comunicação (um *Interprocess Communication - IPC* ), de tal maneira que os dados possam ser transmitidos de uma para outra. Há dois métodos para tal: o *Shared Memory* (compartilhamento de memória); e o *Message Passing* (sistema de passagem de mensagem).

Há várias razões para ter um processo cooperativo.
1. Troca de informações: Compartilhamento de informações interessantes.
2. Aceleração computacional: Dividir uma tarefa em várias pequenas partes.
3. Modularidade: Com as funções do sistema dividido em diferentes processos ou threads.


#### Shared Memory

Esse método torna acess

shared memory and message passing.









Ao ser executado, o processo não é carregado integralmente para a CPU, e sim uma unidade de trabalho chamada de Thread.


Um processo pdoe ser visto
Na memória principal, um processo é um conjunto de quatro sessões



## Threads - criação em `C`

Há duas grandes vantagens
Para multitarefas, os processos nem sempre são uma boa solução.
1. Processos tomam tempo para criar
2. Processos não compartilham dados facilmente


Nas aulas anteriores foram discutidos as vantagens das threads em comparação aos processos. Abaixo está um código em `C` para criação de threads.

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

Capítulo 12 do livro use a cabeça C


Continua em 36 minutos


Diagram of process state

preempção


PCB - process control block
Process representation in linux: PCB em lista encadeada

Ready and wait queues

1h e 8 minutos: representation process scheduling

CPU switch from process to process
Contex-switch time is pure OVERHEAD - A CPU fica ociosa enquanto está trocando de contexto

Process Creation - 1h e 23 min
tree process in linux
process termination - carcading termination - Zombie Orphan
 até 1h e 40 minutos
 volta em 1h e 50 minutos
 
 Comunication Models (Entre processos) - Race Conditions
 
 Producer-Consumer Problem
 -> buffered
 -> unbuffered
 
 shared data
 
 Synchronization
 
 Padrão de projeto observer - javascript

##Complemento
Teoria de filas




