---
layout: post
title: Terraform na pratica - Capitulo 1
image: /img/terraform.png
---


# Instalação no Linux

Antes de mais nada precisamos instalar o Terraform na nossa máquina, 
a versão do linux que eu vou utilizar aqui é o Ubuntu 20.04.
No site do Terraform precisamos baixar o pacote compatível com o nosso 
sistema operacional que, no caso, se encontra 
[nesse link.](https://releases.hashicorp.com/terraform/0.14.6/terraform_0.14.6_linux_amd64.zip)
No momento que escrevo a versão mais atual do Terraform é a 0.14.6!
Por padrão o arquivo que baixamos vem no formato .zip e precisamos descompactar esse arquivo.

Abra o terminal e digite

```
cd ~/Downloads
```


Em seguida vamos utilizar o comando unzip para descomprimir o arquivo:

```
unzip terraform_0.14.6_linux_amd64.zip
```


A saída desse comando deve ser algo parecido com:
```
➜  Downloads unzip terraform_0.14.6_linux_amd64.zip 
Archive:  terraform_0.14.6_linux_amd64.zip
  inflating: terraform 
```

Se olharmos na pasta de Downloads veremos um binário chamado terraform, vamos testar. 
Ainda no terminal execute o seguinte:
```
➜  ./terraform --version
Terraform v0.14.6
```

Agora para evitar de precisarmos sempre passar o endereço completo do arquivo
vamos colocar o arquivo no nosso path:

```
sudo mv terraform /usr/local/bin/terraform
```

O Ubuntu pedirá para digitar a senha de root e por fim vamos testar 
novamente sem passar o endereço completo para termos certeza de que está 
tudo funcionando corretamente:

```
➜  terraform --version
Terraform v0.14.6
```

# Instalação no MacOS

Se você quiser pode seguir o mesmo processo de instalação utilizado no Ubuntu, 
o único ponto de atenção é conferir se está baixando a versão para MacOs.
Apesar de ser a única forma oficial mantida pela Hashicorp, o jeito mais recomendado 
é utilizar o brew. 
Se você tem um Mac e não conhece o comando brew dá uma corridinha lá no site, 
https://brew.sh/index_pt-br , que ele vai te ajudar muito a gerenciar os pacotes no mac!

Para instalar o brew no MacOs basta abrir o terminal e executar um comando:

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```


Agora podemos fazer a complicada tarefa de instalar o terraform executando:
```
brew install terraform
```


Por último vamos confirmar a versão instalada:

```
➜  terraform --version
Terraform v0.14.6
```

# Instalação no Windows

Apesar de todos os nossos exemplos serem executados em um sistema linux, caso você queira acompanhar utilizando o sistema operacional Windows precisará instalar o Chocolatey que ,assim como o Brew pro MacOs, funciona como um gerenciador de pacotes para o Windows.

Acesse o site https://chocolatey.org/ e siga o passo a passo para instalação e em seguida abra o cmd e digite:

```
choco install terraform
```

Para garantir que tudo está funcionando como deveria vamos fazer o seguinte:

```
terraform --version
Terraform v0.14.6
```

# Terraform com Docker


Agora que já vimos como instalar em cada um dos principais sistemas operacionais, vamos brincar um pouco com o Docker. Caso você não conheça o docker, o que eu recomendo, pode instalar o terraform da forma padrão no seu sistema operacional e pular essa parte.    
Sem mais delongas, por que deveríamos utilizar o terraform dentro de um container?
Eu vou te dar um exemplo, enquanto escrevo estamos na versão 0.14.6 do terraform 
e na empresa que eu trabalho praticamente todos os nossos módulos ainda rodam na 
versão 0.13.x. Eu poderia utilizar algum gerenciador de versão 
(falaremos sobre isso mais pra frente), mas caso você goste de fazer backup das suas configs ou
queira alterar inclusive as variáveis que são utilizadas em cada versão, o docker seria a melhor opção. 
O  que eu faço é estipular versões específicas para cada empresa, por exemplo:

A empresa `igordcsouza` usa a versão 0.13.x então quando eu rodar o comando:

```
terraform-igordcsouza --version
Terraform v0.13.x
```

E se eu executar o comando que criei seria algo como:
```
terraform-tnp --version
Terraform v0.14.6
```

E tudo isso sem precisar me preocupar com que pasta eu estou. 
Outro ponto importante são variáveis de ambientes. O Terraform possui algumas 
variáveis de ambiente que ele procura para tentar falar com a Digital Ocean, 
por exemplo. No meu caso, cada comando possui as variáveis corretas setadas, 
então eu não preciso me preocupar de tentar executar um comando do livro e ir 
parar na conta de um cliente.

Vamos testar como isso funciona na prática agora. Caso você não possua o docker 
instalado na sua máquina existem várias formas de instalar, da uma olhada no site 
mas de forma geral você pode utilizar o comando a seguir no seu terminal:

```
curl -fsSL https://get.docker.com | sh;
```

Agora podemos seguir e tentar executar o terraform, sim executar sem instalar nada mais. 
Basta passar o seguinte comando após ter instalado o docker:

```
docker run hashicorp/terraform:latest --version
Unable to find image 'hashicorp/terraform:latest' locally
latest: Pulling from hashicorp/terraform
c9b1b535fdd9: Pull complete 
011000b168e5: Pull complete 
4c096b23c4a8: Pull complete 
Digest: sha256:53fb1c0a78c8bb91c4a855c1b352ea7928f6fa65f8080dc7a845e240dd2a9bee
Status: Downloaded newer image for hashicorp/terraform:latest
Terraform v0.14.6
```


Ainda precisamos ajustar algumas coisas nesse comando para ele ficar utilizável, mas já temos o terraform “instalado” local. 
.Vamos adicionar os seguintes comandos:

* --rm
* -v $(pwd):app
* -w /app/

O comando final deve ficar da seguinte forma:

```
docker run --rm -it -v $(pwd):/app/ -w /app/ hashicorp/terraform:latest
```

Para entender melhor cada um dos parâmetros dê uma olhada na documentação oficial do Docker.
A última coisa que precisamos fazer para utilizar o terraform dentro do container é criar um atalho para esse comando. Seria muito complicado ficar digitando ele inteiro toda hora.
Caso você esteja utilizando o terminal padrão, abra o arquivo com o seu editor preferido, normalmente o caminho é ~/.bashrc, e adicione o seguinte no fim do arquivo:

```
alias terraform='docker run --rm -it -v $(pwd):/app/ -w /app/ hashicorp/terraform:latest'
```


Em seguida recarregue o arquivo reiniciando o terminal ou executando: 

```
source ~/.bashrc
```

Agora podemos utilizar o comando sem maiores problemas:

```
terraform --version
Unable to find image 'hashicorp/terraform:latest' locally
light: Pulling from hashicorp/terraform
Digest: sha256:53fb1c0a78c8bb91c4a855c1b352ea7928f6fa65f8080dc7a845e240dd2a9bee
Status: Downloaded newer image for hashicorp/terraform:light
Terraform v0.14.6
```

Agora se você precisar rodar uma versão diferente do terraform basta trocar o nome do alias por algo similar a terraform-tnp e trocar a versão do container de latest para a desejada e pronto!

## Criando nossa conta na Digital Ocean

Para aprendermos um pouco sobre o Terraform precisamos criar uma conta em algum provedor para executar nossos testes. A DigitalOcean tem um programa de referência que no momento está dando 100$ para cada pessoa que se inscrever pelo link: 

https://m.do.co/c/ee28782de1c9

Para criar uma conta é só seguir o passo a passo no site. Eu recomendo utilizar a integração com o Google ou Github. Mesmo com todo esse crédito, a Digital Ocean exige que você cadastre uma forma de pagamento. Ela pode ser um cartão de crédito ou uma conta no paypal. Com a conta criada, vamos procurar pelo item “API” no menu, provavelmente no canto esquerdo.
A primeira coisa que precisamos fazer nessa tela é gerar um novo token. Coloque um nome pra esse token, clique em salvar. Agora salve esse valor em algum local de fácil acesso pois precisaremos dela mais pra frente.
O próximo passo é cadastrar nossa chave pública na nossa conta para que possamos acessar as máquinas via ssh. Clique na sua fotinha no canto superior direito e no menu clique em Account > Security > Add SSH Key.

Volte para o terminal e verifique se você já possui uma chave configurada, o caminho default é:

```
ls ~/.ssh 
id_rsa  id_rsa.pub  known_hosts
```


Caso você não tenha nada nessa pasta vamos criar agora:

```
ssh-keygen -t rsa
```

E aperte enter para todas as perguntas que serão feitas. Agora se executarmos o mesmo comando você deverá ver dois arquivos `id_rsa` e `id_rsa.pub`.
Queremos o valor do id_rsa.pub para isso vamos executar:

```
cat ~/.ssh/id_rsa.pub
```

Copie a saída do comando e coloque lá na página do digitalocean. Dê um nome que tenha algum significado para essa chave, principalmente se você tiver mais de um notebook. Essa boa prática vai ter ajudar caso precise remover alguma chave por qualquer motivo. 
Clique em Add SSH Key, copie o valor do Fingerprint e salve junto com o token que criamos anteriormente.
