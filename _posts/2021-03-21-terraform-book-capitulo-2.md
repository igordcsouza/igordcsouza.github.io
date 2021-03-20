---
layout: post
title: Terraform na pratica - Capitulo 2
image: /img/terraform.png
---


## Terraform Init

Agora que já temos uma noção de como a estrutura de pasta e arquivos funcionam, vamos conhecer os principais comandos do Terraform. O primeiro comando dos nossos 4 principais é o terraform init.

Para entender na prática o que ele faz vamos criar um arquivo chamado `main.tf` e adicionar o seguinte:

```
provider "digitalocean" {}
```

Vamos falar mais pra frente sobre providers, mas o que você precisa saber nesse momento é que cada provider é como se fosse um dicionário. No nosso caso, estamos baixando um dicionário para ensinar o nosso código a se comunicar com a API do Digital Ocean.

Para manter um tamanho decente, nosso binário não vem com todos os providers por padrão. Então precisamos baixar o nosso plugin antes de executar qualquer outro comando.

Para isso que temos o comando:

```
terraform init
```

Esse comando analisa todo nosso código terraform e encontra todos os `dicionários` declarados e faz download do provider para que possamos utilizar os seus recursos.

Um problema diferenca na versao atual, e que agora precisamos especificar exatamente qual o provedor estamos utilizando.

Abra o terminal e na mesma pasta que criamos o arquivo main.tf execute o comando e verá uma saída parecida com a seguinte:

```
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/digitalocean...

Error: Failed to query available provider packages

Could not retrieve the list of available versions for provider
hashicorp/digitalocean: provider registry registry.terraform.io does not have
a provider named registry.terraform.io/hashicorp/digitalocean

If you have just upgraded directly from Terraform v0.12 to Terraform v0.14
then please upgrade to Terraform v0.13 first and follow the upgrade guide for
that release, which might help you address this problem.

Did you intend to use digitalocean/digitalocean? If so, you must specify that
source address in each module which requires that provider. To see which
modules are currently depending on hashicorp/digitalocean, run the following
command:
    terraform providers
```

Para resolver isso vamos criar um arquivo chamado de `versions.tf` e vamos adicionar o seguinte:

```
 terraform {
    required_version = ">= 0.14, < 0.15"
    required_providers {
        digitalocean = {
            source  = "digitalocean/digitalocean"
        }
    }
}
```

Nesse caso estamos deixando explicito para o terraform que quando utilizarmos o provider `digitalocean` ele deve olhar o provider que se encontra no registry utilizando o path `digitalocean/digitalocean`. Eu nao vou colocar uma versao especifica do provider pois quero usar sempre a ultima disponivel, mas e uma boa pratica especificar uma versao principalmente se voce estiver fazendo uso do codigo em producao. 

Vamos aproveitar pra cadastrar uma versao minima e maxima do terraform, a principio as proximas versoes nao devem ter muitas break changes, mas quando lancarem a 0.15.x eu atualizo esse material!

Vamos rodar novamente o `terraform init`:

```
terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of digitalocean/digitalocean...
- Installing digitalocean/digitalocean v2.5.1...
- Installed digitalocean/digitalocean v2.5.1 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```


Podemos ver que a versão do provider baixada foi a “2.5.1” e agora podemos seguir para conhecer os próximos comandos pois nosso código está pronto para ser “compreendido” pelo terraform.

Um ponto importante aqui e o arquivo `.terraform.lock.hcl`, ele se parece com isso:

```
# This file is maintained automatically by "terraform init".
# Manual edits may be lost in future updates.

provider "registry.terraform.io/digitalocean/digitalocean" {
  version = "2.5.1"
  hashes = [
    "h1:k9itTwJzUpMBTYsXYPoEW/fyoDOcteQc4+OMRmFErbc=",
    "zh:057b8fa0f95213e7d856208d456175335fb673cfef14abf41193f0a2d76e1210",
    "zh:0daee13dd46de95ce2550459942c1433290798bfb5faac12781f81799dd6b05c",
    "zh:13778c00db5c43b2ed5781e2de32d73f34b391c865a52ad3380714bf86251785",
    "zh:2b2bbb1b057c8bf15804a9fd47c30f30b39bcd7ed478bfcad11e221c654f5d02",
    "zh:43284d2b1a356f541723a46219812590d24742558ef4111eda545212fd60f011",
    "zh:6a6e13b55f9aa889e3162d75cb3e585116e8a0d12084629af38f68cdac6aa777",
    "zh:6fa3dbbad99a075768e9449fc6082769da1b76ae31a8e296ae50899835e859a1",
    "zh:79336598d190f511cf3d3323b49081474669d0daa9c1c0d3b21475110ad97bd9",
    "zh:84c4c8d29820229bd94f7d3c5310f1f7208b97e7d4efca2c8e24ae0c0e032267",
    "zh:86926853140d9072986d2cb8ff4693784abd5f5d241b8cec402dfad77d8060ed",
    "zh:95a896f51656b51519b10edf38f11eb766de60297b8551dc0d14a4041dd16d6f",
    "zh:d163da24466cd60eed4749fef56c6593cc6e33be2e210e1b57edfd1c968aa742",
    "zh:e830649afac9e505603002f8a76b2441a0a41c96c6516609e2c07ce0c45f9dc3",
  ]
}
```

Esse arquivo e mantido pelo proprio comando `init` e deve ser adicionado ao seu repositorio. Isso ajuda a manter a versao da mesma forma que voce faria setando uma versao direto no arquivo `versions.tf`.

Caso voce queira atualizar a versao de um provider onde existe esse arquivo, basta executar o comando `terraform init -upgrade=true`, ele vai baixar a ultima versao disponivel do provider sempre respeitando qualquer restricao que voce tenha adicionado. Caso voce tenha mantido como o meu, vai sempre ter a versao mais nova e consequentemente mais instavel.

## Terraform plan

Agora que já rodamos o init e fizemos download da biblioteca da DigitalOcean, vamos fazer uso dos recursos disponíveis para entender melhor como o comando plan funciona. A primeira coisa que iremos fazer é abrir a documentação do terraform na parte que se refere a biblioteca da DigitalOcean (https://www.terraform.io/docs/providers/do/index.html).

Se você prestar atenção vai ver que existem basicamente 3 itens no menu lateral:

* DigitalOcean Provider: Aqui você encontra uma explicação de como configurar corretamente o provider no seu código. Ele explica coisas como quais variáveis ele busca por padrão no seu ambiente e quais parâmetros são aceitos.

* Data Sources: Não é incomum precisar de algum recurso que foi criado em outro código terraform ou mesmo de forma manual, e são nesses momentos que o Data Source pode nos ajudar. Com ele podemos pegar alguns valores de recursos já criados para utilizar no nosso código. Vamos entender mais pra frente com detalhes. 

* Resources: Todos os itens dentro desse menu se referem a como criar e gerenciar recursos diretamente pelo terraform. Ou seja, se queremos criar um droplet e aqui dentro que vamos buscar.

Nesse momento já configuramos o nosso provider, então vamos criar um droplet usando o recurso para isso. Uma dica: apesar da navegação na documentação ser bem simples as vezes é mais rápido buscar no google por “terraform + versão + recurso”. No nosso caso seria: “terraform 0.14 droplet”.

Agora que já encontramos a página vamos copiar exatamente o exemplo disponível na página e colar logo abaixo do nosso provider la no arquivo `main.tf`:

```
resource "digitalocean_droplet" "web" {
  image  = "ubuntu-20-04-x64"
  name   = "web-1"
  region = "nyc2"
  size   = "s-1vcpu-1gb"
}
```

Salve o arquivo e volte para o terminal, nele vamos executar o seguinte comando:

```
terraform plan
```

A saída desse comando deve ser algo mais ou menos assim:


```
terraform plan

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-1"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "nyc2"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

```



O `terraform plan` nunca vai alterar o seu arquivo de estado esteja ele local ou remoto. Mas o que é arquivo de estado? Temos um capítulo inteiro para falar sobre isso, mas por hora entenda que o arquivo de estado é o local onde o terraform armazena toda informação sobre sua infraestrutura. Quando a gente pede pra ele planejar a execução ele usa esse arquivo de estado apenas para comparar o que deveria ser criado com o que ja foi criado. Mas e se eu perder o arquivo? Como eu faço pra usar esse arquivo em time? Calma pequeno gafanhoto, vamos falar sobre isso mais pra frente!


```
  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-1"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "nyc2"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```


Aqui conseguimos ver exatamente tudo que o terraform irá criar para nós com um símbolo de + do lado. Alguns valores do droplet foram pré fixados, como por exemplo a região (region) que já mostra o valor de nyc2, alguns outros valores apenas serão recebidos após a criação do droplet, como é o caso do preço por hora (price_hourly).

Existem duas variações importantes no `plan` que precisamos entender: 
a primeira é o que o terraform chama de update in-place , que basicamente é atualizar um valor de um parâmetro sem afetar o recurso a nível de precisarmos recriar o mesmo. Esse tipo de variação é apresentada da seguinte forma:

```
 An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  ~ update in-place

Terraform will perform the following actions:

  # digitalocean_droplet.web will be updated in-place
  ~ resource "digitalocean_droplet" "web" {
        backups            = false
        created_at         = "2020-04-10T13:38:03Z"
        disk               = 25
        id                 = "188070139"
        image              = "ubuntu-18-04-x64"
        ipv4_address       = "167.172.25.191"
        ipv6               = false
        locked             = false
        memory             = 1024
        monitoring         = false
      ~ name               = "web-1" -> "web-2"
        price_hourly       = 0.00744
        price_monthly      = 5
        private_networking = false
        region             = "nyc3"
        resize_disk        = true
        size               = "s-1vcpu-1gb"
        status             = "active"
        tags               = []
        urn                = "do:droplet:188070139"
        vcpus              = 1
        volume_ids         = []
    }

Plan: 0 to add, 1 to change, 0 to destroy.
```

Enquanto alguns parâmetros podem ser atualizados, alguns precisam que os recursos sejam recriados para que a alteração tenha efeito. No nosso caso poderíamos testar isso modificando a região do nosso droplet.
A saída seria mais ou menos assim:

```
 An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
-/+ destroy and then create replacement

Terraform will perform the following actions:

  # digitalocean_droplet.web must be replaced
-/+ resource "digitalocean_droplet" "web" {
        backups              = false
      ~ created_at           = "2020-04-10T13:38:03Z" -> (known after apply)
      ~ disk                 = 25 -> (known after apply)
      ~ id                   = "188070139" -> (known after apply)
        image                = "ubuntu-18-04-x64"
      ~ ipv4_address         = "167.172.25.191" -> (known after apply)
      + ipv4_address_private = (known after apply)
        ipv6                 = false
      + ipv6_address         = (known after apply)
      + ipv6_address_private = (known after apply)
      ~ locked               = false -> (known after apply)
      ~ memory               = 1024 -> (known after apply)
        monitoring           = false
        name                 = "web-1"
      ~ price_hourly         = 0.00744 -> (known after apply)
      ~ price_monthly        = 5 -> (known after apply)
        private_networking   = false
      ~ region               = "nyc3" -> "nyc1" # forces replacement
        resize_disk          = true
        size                 = "s-1vcpu-1gb"
      ~ status               = "active" -> (known after apply)
      - tags                 = [] -> null
      ~ urn                  = "do:droplet:188070139" -> (known after apply)
      ~ vcpus                = 1 -> (known after apply)
      ~ volume_ids           = [] -> (known after apply)
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

Quando essa situação ocorre, podemos ver exatamente qual o parâmetro está forçando uma recriação pois o terraform colocar uma mensagem “# forces replacements” ao lado desse parâmetro, além de colocar uma mensagem de alerta no próprio recurso avisando que “must be replaced”!

Outro local que podemos ver isso é no fim do comando onde temos um compilado das informações dizendo que destruiremos 1 recurso, criaremos 1 recurso e não alteramos nenhum. Ou seja, no nosso caso vamos destruir o que existe e criar um novo. 

## Terraform Apply

Vamos utilizar o comando `terraform apply` para tentarmos criar nosso droplet.

```
 terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-1"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "nyc2"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value:
```

Se voce prestar atencao, vai perceber que a saida ate aqui e bem similar a que tivemos no `plan`, isso acontece por que o `apply` antes de realmente aplicar as alteracoes ele executa um plan para que voce possa se certificar de que realmente quer fazer isso.

Se voce quiser burlar esse plan e simplesmente aceitar o que vai ser criado, basta executar `terraform apply -auto-approve`.

```
terraform apply -auto-approve
digitalocean_droplet.web: Creating...

Error: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 401 Unable to authenticate you
```

Achou que eu ia te livrar de todos os erros? Achou errado!

Bom, `401 Unable to authenticate you` significa que nao foi possivel se autenticar com a api da Digital Ocean. Se voltarmos la na pagina sobre o provider vamos conseguir entender melhor como corrigir o erro.

De toda forma, a primeira coisa que precisamos fazer e criar um token no painel da Digital Ocean. Acesse o painel de controle da sua conta e procure por API, no momento que escrevo e o ultimo item da coluna esquerda. Clique em `generate new token` de um nome para o seu token e copie o token que vai aparecer para voce. Algo como:

```
 b783123asd294f002c5ea9af1231298327903284kmmsadoiu48d02707a2018588d
```

Agora vamos seguir o jeito mais simples e completamente 

```
variable "do_token" {
  default = "b783123asd294f002c5ea9af1231298327903284kmmsadoiu48d02707a2018588d"
}

provider "digitalocean" {
  token = var.do_token
}
```


O nome `do_token` poderia ser qualquer coisa, esse e o campo onde voce nomeia sua variavel e o parametro `default` e onde voce cadastra um valor padrao pra ela. Sem esse parametro, toda vez que voce rodar o `plan` ou o `apply` ele vai te perguntar o valor da variavel. Se isso e o que voce quer, basta remover a linha com o parametro default.


Vamos executar novamente nosso apply:
```
terraform apply -auto-approve
digitalocean_droplet.web: Creating...

Error: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 401 Unable to authenticate you
```

Mesmo com o token cadastrado, a gente continua recebendo o mesmo erro. Isso acontece por que apesar de termos criado uma variavel nao contamos para o provider que ele deve utilizar essa variavel. Vamos fazer isso agora.

```
 provider "digitalocean" {
  token = var.do_token
}

variable "do_token" {
  default = "b783726c929c48febf8555eee856f84f002c5ea9af7d49648d02707a2018588d"
}

resource "digitalocean_droplet" "web" {
  image  = "ubuntu-20-04-x64"
  name   = "web-1"
  region = "nyc2"
  size   = "s-1vcpu-1gb"
}
```

Agora o provider `digitalocean` sabe que ele deve procurar por uma variavel chamada `do_token` e utilizar no parametro `token`.

Vamos tentar mais uma vez criar nossa maquina.

```
terraform apply -auto-approve
digitalocean_droplet.web: Creating...

Error: Error creating droplet: POST https://api.digitalocean.com/v2/droplets: 422 nyc2 is unavailable.
```

Opa, agora aconteceu uma das maiores felicidades de quem escreve codigo. Mudou o erro! O que e bom pois antes estavamos recebendo uma mensagem de que nao que nao conseguiamos nos conectar e agora a mensagem foi `422 nyc2 nao esta disponivel`. Acontece que apesar da documentacao do provider dar o exemplo com a regiao `nyc2`, essa regiao nao esta disponivel a algum tempo ja. Eu nao sei exatamente se eles nunca se preocuparam em corrigir isso ou se o objetivo e fazer a gente aprender a debugar o problema. Enfim, a solucao e so alterar a regiao para qualquer outra que exista, vamos mudar pra `ams3` que e a regiao de Amsterdam pra ficar mais pertinho de mim, mas voce pode usar outra qualquer como `nyc3` por exemplo.

Mais uma tentativa:

```
terraform apply -auto-approve

digitalocean_droplet.web: Creating...
digitalocean_droplet.web: Still creating... [10s elapsed]
digitalocean_droplet.web: Still creating... [20s elapsed]
digitalocean_droplet.web: Still creating... [30s elapsed]
digitalocean_droplet.web: Creation complete after 33s [id=232247846]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Muito bom, mas agora vem a questao! Como a gente pode ter alguns dados desse droplet que acabamos de criar? Por exemplo, se eu quiser saber o IP desse droplet, como a gente faz?

Pra isso vamos utilizar os `outputs`.

## Terraform Output

Toda documentacao de um resource e dividido em pelo menos tres niveis. 

* Exemplo de Uso
* Referencia de Argumento (ou parametro)
* Referencia de Atributos

Todos os itens que tivermos dentro do nivel de referencia de atributo podem ser utilizados seja para passar para um outro resource ou para ser utilizado como output do nosso codigo. Antes de comecarmos a misturar atributos de um recurso com argumento de outro, vamos usar os atributos na sua forma mais simples, como um output.

O `output` em si e uma feature que permite voce apresentar o valor de alguma coisa do seu codigo na tela. Mais pra frente vamos ver uma outra utilidade do `output` que e retornar os valores de um modulo. Mas fica tranquilo que isso e mais pra frente.

Por hora vamos adicionar o seguinte num arquivo chamado `output.tf`, que esteja na mesma pasta que nosso `main.tf`.

```
output "droplet_ip" {
  value = "hashicourse"
}
```

Vamos executar novamente o comando `apply` e vamos ver que agora temos mais um retorno no nosso codigo.

```
terraform apply -auto-approve
digitalocean_droplet.web: Refreshing state... [id=232247846]

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

droplet_ip = "hashicourse"
```

Nada foi criado ou alterado, como esperado, mas vemos um `Outputs:` na resposta e o valor que a gente cadastrou pro ip do nosso droplet. Agora manter esse valor hardcoded nao vai nos ajudar muito. Vamos entao utilizar um atributo do nosso droplet pra isso. 

Primeiro, deixa eu te mostrar o codigo:

```
 resource "digitalocean_droplet" "web" {
  image  = "ubuntu-20-04-x64"
  name   = "web-1"
  region = "ams3"
  size   = "s-1vcpu-1gb"
}

output "droplet_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```

.A forma de chegarmos nesse atributo e bem simples.
* [A] Primeiro voce precisa pegar o nome do recurso, que e aquele primeiro ali "digitalocean_droplet" que nada mais e como uma forma de dizer para o provedor qual o recurso voce ta querendo criar. 
* [B] Segundo pega o nome que voce deu para aquele recurso, no nosso caso "web". E muito comum precisarmos utilizar mais de uma vez um recurso no nosso codigo e essa e uma das formas de nomear aquele recurso. Vale lembrar que esse nome e unico, ok?
* [C] Por ultimo precisamos ir la na documentacao e ver qual dos atributos disponiveis nos fornece o que precisamos. E raro que voce precise de algo que nao esta ali disponivel, mas raro nao e impossivel.

Por fim basta montar sempre no formato A.B.C :) 

Se voce rodar o seu codigo agora deve ter um retorno como o seguinte:

```
terraform apply -auto-approve
digitalocean_droplet.web: Refreshing state... [id=232247846]

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

droplet_ip = "104.248.203.68"
```

Muito provavelmente seu numero de ip é diferente do meu, mas se por algum milagre voce pegou o mesmo, bate um print e me manda :)

Uma ultima dica no que se refere a outputs e que voce nao precisa utilizar apenas o apply para ter acesso a todos os outputs do seu codigo. Considerando que voce ja tenha executado o apply pelo menos uma vez, voce pode utilizar o `terraform output`:

```
terraform output
droplet_ip = "104.248.203.68"
```

## Terraform Destroy

Agora que ja criamos um droplet, voce talvez esteja se perguntando "Beleza, agora como eu desligo esse droplet pra nao gastar meu bonus?", bom pra isso tem um comando com nome bem sugestivo `destroy`.
O `terraform destroy` e o oposto do `apply`. Uma das coisas que ambos tem em comum e o plan antes de executar. A gente nao falou antes, mas existe um parametro no `plan` chamado de `-destroy`. E esse e o parametro que o terraform usa quando a gente executa o `terraform destroy`, a saida deve ser parecida com a seguinte:

```
terraform destroy

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # digitalocean_droplet.web will be destroyed
  - resource "digitalocean_droplet" "web" {
      - backups              = false -> null
      - created_at           = "2021-02-17T19:34:37Z" -> null
      - disk                 = 25 -> null
      - id                   = "232254786" -> null
      - image                = "ubuntu-20-04-x64" -> null
      - ipv4_address         = "104.248.197.79" -> null
      - ipv4_address_private = "10.110.0.2" -> null
      - ipv6                 = false -> null
      - locked               = false -> null
      - memory               = 1024 -> null
      - monitoring           = false -> null
      - name                 = "web-1" -> null
      - price_hourly         = 0.00744 -> null
      - price_monthly        = 5 -> null
      - private_networking   = true -> null
      - region               = "ams3" -> null
      - resize_disk          = true -> null
      - size                 = "s-1vcpu-1gb" -> null
      - status               = "active" -> null
      - urn                  = "do:droplet:232254786" -> null
      - vcpus                = 1 -> null
      - volume_ids           = [] -> null
      - vpc_uuid             = "b83009d3-8cb0-4545-8123-6f2238e40284" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Changes to Outputs:
  - droplet_ip = "104.248.197.79" -> null

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value:
```

O que vemos aqui e que vamos destruir todos os nossos recursos, que no caso e apenas um droplet. Voce pode digitar `yes` no comando acima ou utilizar o `-auto-approve` junto com o destroy para destruir todos os recursos sem nem olhar pro `plan -destroy`.

```
terraform destroy -auto-approve
digitalocean_droplet.web: Destroying... [id=232247846]
digitalocean_droplet.web: Still destroying... [id=232247846, 10s elapsed]
digitalocean_droplet.web: Still destroying... [id=232247846, 20s elapsed]
digitalocean_droplet.web: Destruction complete after 22s

Destroy complete! Resources: 1 destroyed.
```

Beleza, agora voce pode descansar em paz que seu bonus/dinheiro nao esta sendo mais consumido. Lembre-se sempre de destruir os recursos da sua conta pra garantir que nao tenha surpreza no fim do mes. Importante também e criar um alerta la na digitalocean, eu recomendo criar um valor de 5$ que e o custo de um droplet, se passar disso provavelmente algo esta errado. 

## Arquivo de Variaveis

Uma coisa comum de se ver sao codigos que utilizam uma variavel da seguinte forma:

```
 variable "do_token" {
  description = "Uma descricao bacana!"
  type        = string
}
```

Ou seja, nao possuem um valor padrao para a variavel, acontece que dependendo da quantidade de variaveis que existem nesse formato, pode ser bem chato de ficar colocando uma a uma. 

Uma solucao, bem gambiarra, que eu ja fiz muito e colocar um valor `default` para cada uma dessas variaveis e garantir que nao iria adicionar as alteracoes desse arquivo no repositorio git. O que normalmente acabava acontecendo depois de um tempo. Acontece que o `terraform` te da a possibilidade de usar um arquivo so para adicionar esse valores. Basta criarmos um arquivo `qualquer_coisa.tfvars`. Normalmente as pessoas utilizam `terraform.tfvars` ou entao `nome_do_ambiente.tfvars`. Vamos criar o nosso como `terraform.tfvars` mesmo.

```
 do_token = "b783123asd294f002c5ea9af1231298327903284kmmsadoiu48d02707a2018588d"
```

Esse e o formato do arquivo de variaveis, caso exista um valor padrao e voce queira sobreescreve ele, pode colocar o novo valor nesse arquivo e vai funcionar também. Agora precisamos dizer para o `terraform` que ele deve buscar os valores das variaveis primeiro nesse arquivo, para isso vamos executar o `apply` novamente passando o `-var-file` como parametro. Esse `-varfile` vale para o `plan` e o `destroy` também. 

```
terraform apply -var-file terraform.tfvars

An execution plan has been generated and is shown below. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:
  # digitalocean_droplet.web will be created
  + resource "digitalocean_droplet" "web" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "web-1"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "ams3"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }
Plan: 1 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + droplet_ip = (known after apply)

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

digitalocean_droplet.web: Creating...
digitalocean_droplet.web: Still creating... [10s elapsed]
digitalocean_droplet.web: Still creating... [20s elapsed]
digitalocean_droplet.web: Still creating... [30s elapsed]
digitalocean_droplet.web: Creation complete after 34s [id=232397944]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

droplet_ip = "206.189.0.61"
```
