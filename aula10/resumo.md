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


#### Hierarquia 
