---
tags: hashicorp, consenso, consul, raft
layout: post
title: Introdução ao protocolo de Consenso (RAFT) no HashiCorp Consul
image: /img/consul.png
---

# O que é o RAFT?
O RAFT faz parte de uma categoria de algoritmos chamados de Consensus Algorithm (Algoritmos de Consenso). Vamos entender o que são esses algoritmos antes de nos aprofundarmos no RAFT propriamente dito. 

# O que e um algoritmo de consensus?
Consenso é problema fundamental para tolerância a falhas em sistemas distribuídos. Basicamente o problema em questão consiste em múltiplos servidores concordando (entrando em consenso). Um algoritmo de consenso típico funciona quando a maioria dos servidores se encontra disponível. Por exemplo, um cluster de 5 servidores consegue funcionar normalmente mesmo se dois dos seus servidores pararem de funcionar, isso acontece por que se mantem a maioria dos servidores funcionando. Ainda que mais servidores venham a falhar, por exemplo se 4 dos nossos 5 servidores falharem, ainda assim o servidor que sobrou sempre retornara resultados corretos, apesar de não conseguir mais avançar com novos valores. 

Esses algoritmos normalmente são utilizados no contexto de replicar maquinas de estado. Se voce fez faculdade alguma faculdade na area de computação muito provavelmente ouviu falar um pouco sobre maquina de estado. No caso, cada servidor possui uma maquina de estado e um log. A maquina de estado e o componente que queremos que seja tolerante a falhas, como por exemplo uma tabela de hashes. Para os servidores clientes vai parecer que eles estão se comunicando apenas com uma única maquina de estado, mesmo que a minoria dos servidores rodando os agentes em modo server falhem. 

Cada maquina de estado recebe como entrada um comando que vem do seu log. No exemplo usando uma tabela de hashes, o log poderia incluir comandos como por exemplo x é igual a 3. O algoritmo de consenso e utilizado para concordar, e de certa forma garantir, nos comandos que estão nos logs dos servidores. O que o algoritmo deve fazer nesses casos e garantir que se qualquer uma das maquinas de estado, os nossos agentes rodando em modo server, adicionar um comando “x = 3” como o comando na posição 10 do log, por exemplo, nenhum outro servidor poderá adicionar um comando diferente na posição 10.  O resultado disso e que cada maquina de estado tem os mesmos comandos na mesma ordem produzindo assim os mesmos resultados e chegando no mesmo estado.

O RAFT e um algoritmo que foi projetado para ser simples de entender. Ele é equivalente a um outro protocolo muito conhecido chamado Paxos, tanto em tolerância a falhas quanto em performance. O RAFT separou os principais elementos do protocolo como eleição de um líder, replicação e logs, segurança e com isso foi possível reduzir o numero de estados que devem ser considerados. 

Como dito anteriormente, o RAFT e bem parecido com outros, vamos dar uma olhada em tres coisas que ele se destaca um pouco alem dos outros:

**- Simplicidade:** Primeiro vamos deixar claro ser mais simples que os outros algoritmos não torna ele trivial, e tudo bem se voce nao entender ele por completo. Mas existem estudos que comprovam que aprender o RAFT e bem mais simples do que Paxos. E sua implementação, que infelizmente não vamos tratar aqui, também e bem mais simples.

**- Leader mais forte:** RAFT usa uma forma de liderança diferente dos outros algoritmos. Por exemplo, entradas no log só são possíveis no líder e o fluxo segue do líder para os outros servidores. Isso simplifica bastante a gestão da replicação de logs.

**- Eleição do Líder:** A eleição e feita utilizando um cronômetro randômico. Isso adiciona um mecanismo simples ao heartbeat que e uma funcionalidade utilizada por qualquer algoritmo de consenso, enquanto resolve os conflitos durante a eleição de forma simples e rápida.

**- Mudança nos Membros:** Para mudar os membros de um cluster, o algoritmo utiliza um novo método de junção, onde a maioria entre duas configurações se sobrepõem. Isso permite que o cluster continue operando normalmente durante mudanças de configurações.

Antes de prosseguirmos para entender como o Consul faz uso desse protocolo, temos alguns termos que precisamos ter em mente.

**- Log:** É o principal unidade de trabalho no sistema RAFT. O problema de consistência pode ser dividido em logs replicados. Como logs, entenda uma sequencia de entradas ordenadas por chegada. Essas entradas podem ser qualquer alteração no cluster. Consideramos que o log esta consistente se todos os membros concordam com as entradas e sua ordem.

**- FSM (Maquina de estado finita):** Uma FSM é uma coleção de estados que podem ser alterados entre eles. Quando novos logs são recebidos as maquinas de estado podem alterar seus estados. Vale ressaltar que a aplicação da mesma sequencia de logs deve sempre resultar no mesmo estado. Ou seja, o comportamento deve ser determinístico. 

**- Peer set (Conjunto de Pares):** Chamamos de conjunto de pares o conjunto de todos os membros participando na replicação de logs. Se referindo a como o Consul trata o conjunto de pares, estamos falando apenas dos servidores que possuem agentes rodando em modo server. 

**- Quórum:** Chamamos de quorum a maioria dos membros de um conjunto de pares. Imaginando um conjunto de 10 servidores, o nosso quorum seriam 6 servidores. A conta e feita no formato de (10/2)+1 ou de forma mais genérica (N/2)+1.

**- Committed Entry (Entrada Comprometida):** Uma entrada ou registro e considerada committed quando e armazenada em um quorum. Por exemplo, no caso onde temos 10 servidores assim que 6 deles tiverem a mesma entrada podemos dizer que ela e uma entrada comprometida. Essa tradução parece não ser das melhores mas pense em comprometida como alguém que esta comprometida a fazer algo. Como voce provavelmente se comprometeu a aprender mais sobre Consul por algum motivo :)

**- Líder:** Provavelmente o mais fácil de se entender, esse e o servidor Líder. Apenas um servidor pode ser o Líder por vez. O Líder é responsável por receber as entradas nos logs, replicar para os outros servidores do cluster e gerenciar quando uma entrada é considerada como comprometida.

Todos os agentes em modo server sempre estão em um dos seguintes estados: Follower (Seguidor), Candidate (Candidato) ou Líder.
Os servidores iniciam em modo seguidor. Nesse modo eles aceitam entradas de log vindo de um líder e tem direito de votar. Se nenhuma entrada for recebida durante algum tempo esses servidores podem se auto-promover para o estado de candidato. Nesse estado os servidores solicitam uma votação dos seus pares, se um candidato receber voto do quorum, então ele será promovido a líder. Como líder ele deve aceitar novos registros de log e replicar esses registros para os seguidores. 

Quando o líder recebe um novo registro, ele armazena em um local durável e tenta replicar para o quorum de seguidores. Lembra que falamos sobre uma configuração com quorum tomar o lugar da configuração anterior do cluster?  
Quando esse log e considerado como comprometido ele pode ser convertido para uma maquina de estado finita. Esses estados finitos são específicos de aplicação para aplicação, no caso do Consul é utilizado o MemDB para manter o estado do cluster.

# Raft no Consul

Antes de mais nada, caso não tenha ficado claro anteriormente, precisamos ter em mente que apenas os agentes em modo server participam do protocolo RAFT no cluster. Todos os agentes em modo cliente simplesmente fazem um redirecionamento das requisições para um server. 

Quando iniciamos um Consul cluster utilizando apenas um servidor, precisamos utilizar o método bootstrap que serve para permitir que o servidor se auto eleja líder. Quando um líder é eleito, outros servidores podem ser adicionados como pares para preservar a consistência. Eventualmente, quando mais servidores tiverem sido adicionados podemos desabilitar esse modo e passar a utilizar o modelo padrão do raft.

Por causa do tipo de replicação do RAFT, a performance e muito dependente da latência de rede. Por essa razão, cada datacenter elege um leader independente e mantem um conjunto de pares diferentes. Os dados são divididos por datacenter, assim cada líder e responsável apenas pelo seu próprio datacenter. Por datacenter entenda um cluster do consul. Podemos ter umas arquitetura multi-cluster que também pode ser chamada de multi-datacenter. 

# Modos de Consistência 


Apesar de todas as escritas serem replicadas através do RAFT, leituras podem ser mais flexíveis. O Consul suporta tres diferentes modos de consistência para leitura:

**- default:** RAFT faz uso de uma especie de alocação de líder, provendo uma janela de tempo onde o líder assume seu papel de forma estável. De toda forma, se o líder for separado dos demais pares um novo líder sera eleito enquanto o antigo continua na sua alocação. Ou seja, por algum tempo teremos 2 servidores no estado de líder. Mas ainda assim não existe um risco de split-brain já que lideres antigos serão incapazes de adicionar novos registros nos logs do cluster. Entretanto, se for requerido uma leitura no líder antigo, os valores fornecidos por ele podem ser potencialmente obsoletos. Esse modo depende apenas nessa alocação do líder, potencialmente expondo seu clientes a receberem valores obsoletos. A decisão de utilizar esse modo se da por que apesar desses possíveis problemas, leituras são normalmente rápidas e normalmente bem consistente. Existe sim a possibilidade do erro, mas a janela desse erro e bem pequena já que assim que o líder for separado dos seus pares ele mesmo deve se afastar desse papel por conta da segregação.

**- consistent:** Esse modo é extremamente consistente, ele requer que o líder verifique com um quorum de pares se ele ainda e o líder. O que adiciona mais uma chamada entre os servidores, fazendo com que a latência aumente mas em contra-partida aumentando a consistência dos dados. Importante entender dentro do seu cenário qual a melhor opção, no geral o modo default e o mais indicado, mas se pra voce a consistência for extremamente mais importante, vale a pena alterar o modo para consistent e analisar como seu cluster vai performer. Claro, isso se voce tiver um bom monitoramento do seu cluster. Caso você já tenha um cluster em produção e esteja procurando por esse tipo de aprimoramento, mas não tem um monitoramento pronto, fica tranquilo que vamos falar sobre isso mais pra frente.

**- stale:** A tradução da palavra stale seria velha, mas acho que podemos trabalhar com obsoleto pra fazer um pouco mais de sentido. Esse modo permite que qualquer servidor sirva uma leitura, independente se ele é o líder ou não . Isso quer dizer que leituras podem ser arbitrariamente obsoletas. O lado positivo desse modo e que a leitura é extremamente rápida e a escalabilidade de leituras e muito grande, por outro lado os valores são obsoletos. O que não  necessariamente quer dizer que sejam errados, mas podem ser. 

<hr>

- https://www.usenix.org/system/files/conference/atc14/atc14-paper-ongaro.pdf
- https://raft.github.io
- https://www.consul.io/docs/architecture/consensus
