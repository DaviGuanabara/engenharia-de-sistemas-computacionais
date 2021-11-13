### DNS: Domain Name System

Quando um usuário acessa um *site* na Internet, o mesmo digita um nome, por exemplo:

              `www.google.com.br`
              
              
Sabendo que o sistema de endereçamento utilizado é baseado no TCP/IP, como o navegador é capaz de converter o nome digitado no endereço de IP (*Internet Protocol*) do servidor requisitado ?
Essa tradução é feita pelo *Domain Name System* (DNS).

O DNS é um protocolo da camada de aplicação que roda em UDP (usando a porta 53) no qual fornece o serviço de tradução de *hostnames* (nome dos servidores) em IP *address*. Seu funcionamento é baseado em um sistema de banco de dados distribuidos implementado em uma hierarquia de servidores. 


Essa arquitetura fora adotada, em contraste com o *design* centralizado, por evitar:

1. Único ponto de falha
2. Volume de tráfego
3. Manutenção
4. Distância do usuário (tempo de resposta)
5. Baixa escalabilidade


#### Hierarquia e funcionamento


A hierarquia de servidores de DNS é comporta por:

0. *Local*: Não pertence a hierarquia, mas é de suma importância. É proporcionado pelo *Internet Service Provider* (IPS, debatido em aulas anteriores) 
1. *Root*: cópia de 13 servidores, gerenciados por 12 diferentes organizações, e coordenado pela Internet Assigned Numbers Authority (IANA)
2. *top-level domain* (TLD): fornece o IP *address* para os *authoratives servers*. É mantido por diferentes empresas, como o Verisign Global Registry (*.com*) e Educause (*.edu*)
3. *Authoritative*: toda organização com acesso público deve prover servidores DNS (que pode ser mantido por eles ou por terceiros), com o *primary* sendo o servidor principal e o *secondary* o de *backup*.

Voltando ao exemplo do `www.google.com.br`, quando um usuário digita esse *hostname* e aperta enter, o navegador (*browser*) gera uma uma estrutura de dados, chamada de DNS *query* *message* (ou mensagem de consulta DNS), com uma série de informações, como o nome `www.google.com.br`, que serão enviadas para o *local* DNS *server*.


A partir do *local* DNS *server*, pode ocorrer duas formas de interação: recursiva e iterativa.

##### Iterativa

No modelo iterativo, o *local server* envia um *request* para cada um dos servidores da hierarquia, como mostrado na Figura 01:

1. Usuário: *request* para *local server*
2. *local server*: *request* para o *Root server*
3. *local server*: *response* do *Root server*
4. *local server*: *request* para o TLD *server*
5. *local server*: *response* do TLD *server*
6. *local server*: *request* para o *Authorative server*
7. *local server*: *response* do *Authorative server*
8. Usuário: *response* do *local server*

Figura 01: Modelo Iterativo \
![image](imagens/modelo%20iterativo.png)


##### Recursivo

No modelo recursivo, a sequência de *requests* ocorrem em cadeia entre os servidores:


1. Usuário: *request* para o *local server*
2. *Local server*: *request* para *root server*
3. *Root server*: *request* para TLD *server*
4. TLD *server*: *request* para *Authorative server*
5. TLD *server*: *response* do *Authorative server*
6. *Root server*: *response* do TLD *server*
7. *Local server*: *response* do *root server*
8. Usuário: *response* do *local server*


Figura 02: Modelo Recursivo \
![image](imagens/modelo%20recursivo.png)


##### DNS *caching*

Nos dois casos apresentados anteriormente, foi necessário 8 DNS *query messages* para a execução do serviço de tradução de *hostname* para IP *address*. Porém, de forma a minimizar os impactos de tantas requisições, o *local server* utiliza um sistema de armazenamento temporário (pois o mapeamento entre *hostname* e IP *address* não é permanente, alterando-se com o tempo) da tradução, fazendo com o número de DNS *query messages* caia para apenas 2, o *request* e o *response* entre o usuário e o *local server*.


##### Resource Records


O DNS é um banco de dados distribuidos que armazena uma tupla de 4 elementos, *Name*, *Value*, *Type* e *time to live* (TTL), o qual é utilizado para prover, entre outros, o mapeamento *hostname* para IP *address*. O dado armazenado em cada um dos 2 primeiros elementos citados da tupla variam conforme o *type*. O último elemento especifica quando o *resource record* deve ser removido do *cache*.

A seguir será mostrado os dados contidos em *Name* e *Value* para cada caso de *Type* (os exemplos mostrados não contém o TTL).

*Type* =

1. A: *Name* = *hostname*, *Value* = IP *address* `(relay1.bar.foo.com, 145.37.93.126, A)` 
2. NS: *Name* = *domain*, *Value* = *Authorative hostname server* `(foo.com, dns.foo.com, NS)` 
3. *CNAME*: *Name* = apelido (*alias*) para *hostname* , *Value* = *hostname* `(foo.com, relay1.bar.foo.com, CNAME)` 
4. MX: *Name* = apelido (*alias*) do *mail hostname*, *Value* = *mail hostname* `(foo.com, mail.bar.foo.com, MX)`


##### DNS *Messages*

Figura 03: Formato do DNS *message* \
![image](imagens/DNS%20message%20format.png)


O formato da mensagem DNS é dado na Figura 03


