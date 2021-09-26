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




