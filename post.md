> English version of the post [here](https://www.airpair.com/posts/review/5516f8187fb1af1100e5a297)

> Nota: o post também está disponível no Medium [aqui](https://medium.com/@heitorburger/introdu%C3%A7%C3%A3o-ao-docker-para-o-desenvolvedor-node-js-f58e1630065f)

**Nível de dificuldade:** Iniciante

**Requisitos:** Mac OS X (Este tutorial assume que você está usando um Mac, mas você pode encontrar instruções de instalação para [Windows](https://docs.docker.com/installation/windows/) ou [Ubuntu](https://docs.docker.com/installation/ubuntulinux/) e pular a seção de _Instalação_)

A empresa [Docker](https://www.docker.com/) acabou de celebrar o seu [segundo aniversário](https://blog.docker.com/2015/03/dockers-2nd-birthday-wishes-qa-with-solomon-hykes-founder-of-docker/), mas o que eles criaram ainda é uma “nova” tecnologia. Um monte de amigos desenvolvedores com quem eu falo já ouviram ou leram sobre a tecnologia Docker, mas ainda não o usam, ou nem testaram. Docker permite que você faça coisas muito legais como testar rapidamente o seu app em desenvolvimento, com exatamente o mesmo ambiente como de QA / Teste / Produção, ou compartilhar sua aplicação com outros desenvolvedores para um rápido onboarding. A analogia comumente usado para **Docker é compará-la a containers reais ou peças de LEGO**: **ele fornece uma unidade fundamental**, e com ele uma maneira de que **uma aplicação** seja **portátil e móvel**, independentemente do hardware.

Neste tutorial, eu vou dar uma visão geral do que é o Docker e por que você pode ter interesse em usá-lo, como instalá-lo, e depois vamos trabalhar na criação de um container Node e de um app express template dentro dele. **Este é um tutorial longo**! O tutorial oficial “Getting Started” do Docker é mais rápido e sucinto que este, o que pretendo fazer aqui é **explicar o que está acontecendo em cada passo ao longo do caminho**.

**O que nós vamos cobrir:**

*   Introdução (O que é Docker e por que usá-lo)
*   Instalação
*   Dockerfiles e o Docker Hub
*   Docker Pull: Baixando uma imagem Ubuntu
*   Docker Run: Executando nossa imagem Ubuntu e acessando um container
*   Docker Commit: Instalando node, npm, express e fazendo "commit" de nossas alterações
*   Docker Push: Fazendo "push" do nosso container para que outras pessoas possam utiliza-lo

#### Notas:

Eu irei referenciar comandos executados em seu próprio terminal como:

```bash,linenums=true
$ command
```

E comandos dentro de containers como:

```bash,linenums=true
$ root: command
```

### Introdução

Você provavelmente já ouviu falar de Docker. Todos os dias há uma notícia na primeira página do HackerNews sobre ele, ou você vê menções no Twitter / IRC. Sua popularidade cresceu muito nos últimos dois anos, e diversos provedores de cloud já o suportam. Se você está curioso sobre Docker, mas ainda não teve tempo de testa-lo, este tutorial é para você. ☺

Ok, então o que é Docker? Bem, _Docker_ pode ser uma referência a algumas coisas:

*   **Docker client:** isto é o que está sendo executado em nossa máquina. É o binário do Docker que estaremos interagindo sempre que abrirmos um terminal e executarmos `$ docker pull` ou `$ docker run`. Ele se conecta ao Docker daemon que faz todo o trabalho pesado, seja no mesmo host (no caso do Linux) ou remotamente (no nosso caso, interagindo com a nossa VirtualBox VM).
*   **Docker daemon:** isso é o que faz o trabalho pesado de construção, execução e distribuição de seus containers Docker.
*   **Docker Images:** imagens do Docker são as "blueprints" para nossas aplicações. Mantendo a analogia dos containers/peças de Lego, eles são nossas "plantas" (como em arquitetura, não biologia) para realmente construir um exemplo real e funcional. Uma imagem pode ser um sistema operacional como o Ubuntu, mas também pode ser um Ubuntu com o seu aplicativo web e todos os seus pacotes necessários instalados.
*   **Docker Container:** containers são criados a partir das imagens do Docker, e eles são as instâncias reais de nossos containers / peças de Lego. Eles podem ser iniciados, executados, parados, deletados, e movidos.
*   **Docker Hub (Registry):** o [Docker Registry](https://github.com/docker/docker-registry) é um registro hospedado em um servidor que pode conter imagens do Docker. Docker (a empresa) oferece um Docker Registry público chamado de Docker Hub que vamos utilizar neste tutorial para baixar algumas imagens, mas eles oferecem todo o sistema open-source para que pessoas possam ter seus próprios servidores e armazenar imagens privadas.

Agora que esclarecemos algumas partes do Docker, aqui está uma lista de razões pelas quais você pode querer usá-lo:

*   Simplificar configuração de um ambiente de desenvolvimento
*   Testar rapidamente a sua aplicação em um ambiente semelhante ao de QA / Teste / Produção (overhead menor em comparação com VMs)
*   Compartilhar sua app + ambiente com outros desenvolvedores, o que permite um onboard rápido / confiável.
*   Capacidade de realizar diff em containers (pode ser imensamente útil em debugging)

### Instalação

A execução de um container, e, portanto, Docker, requer uma máquina Linux. Já que nós estamos usando um Mac, isso significa que vamos precisar de uma VM. Para tornar o processo de instalação mais fácil, podemos usar o Boot2Docker que instala a ferramenta de gerenciamento Boot2Docker, o VirtualBox, e configura uma máquina virtual dentro dele com Docker já instalado. ☺

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*HOqvVFhyxu52LxzLrL6z6Q.png" alt="boot2docker"/>
</p>

Acesse este link para baixar a versão mais recente do Boot2Docker (Boot2Docker-1.5.0.pkg no momento que este post foi escrito):

[https://github.com/boot2docker/osx-installer/releases/latest](https://github.com/boot2docker/osx-installer/releases/latest)

Após a instalação, vá até a pasta "Aplicativos" e abra o Boot2Docker. Isso vai abrir um novo terminal e executar alguns comandos que basicamente iniciam uma VM já com Docker instalado, dentro do VirtualBox, e em seguida, define algumas variáveis de ambiente para que possamos acessar essa VM do nosso terminal. Se você não quer sempre abrir o Boot2Docker para interagir com Docker, basta executar os seguintes comandos:

```bash,linenums=true
# Creates a new VM if you don't have one  
$ boot2docker init

# Starts the VM  
$ boot2docker start

# Sets the required environment variables  
$ $(boot2docker shellinit)
```

Agora execute:

```bash,linenums=true
$ docker run hello-world
```

Isso vai fazer o Docker baixar a imagem hello-world do Docker Hub e iniciar um container com base nele. O terminal deve mostrar um output que diz:

```bash,linenums=true
Hello from Docker.  
This message shows that your installation appears to be working correctly.
```

Uhul! Docker está instalado.☺

(Se você tiver qualquer problema, fique a vontade para entrar em contato comigo ou você pode encontrar as instruções de instalação oficiais da Docker [aqui](https://docs.docker.com/installation/mac/))

### Dockerfiles e o Docker Hub

Antes de seguir em frente, eu acho que é importante entender o que aconteceu quando nós executamos `$ docker run hello-world` para que você não esteja simplesmente copiando+colando os próximos comandos. `docker run` é o comando básico que usamos para iniciar um container com base em uma imagem, ao mesmo tempo que passamos comandos para ele. Neste caso, nós dissemos, “Docker, execute um container com base na imagem hello-world, sem comandos extras”. Em seguida, ele fez o download da imagem no Docker Hub e começou um container dentro da VM no VirtualBox com base nessa mesma imagem. Mas de onde vem o hello-world? É aí que Docker Hub entra. O Docker Hub, como dissemos na introdução, é o registro público, contendo imagens de containers para serem usadas com o Docker, criadas pelo Docker, outras empresas e indivíduos. Aqui você pode encontrar a imagem hello-world que acabamos de executar:

[Docker Hub Hello-World Image](https://registry.hub.docker.com/u/library/hello-world/)

Toda imagem é construída usando um Dockerfile, ou a partir de um container. Na descrição da imagem hello-world, você pode encontrar um [link para o seu Dockerfile](https://github.com/docker-library/hello-world/blob/master/Dockerfile) que só tem 3 linhas:

```
FROM scratch  
COPY hello /  
CMD [“/hello”]
```

Dockerfiles são apenas arquivos de texto contendo instruções para o Docker sobre como construir a imagem de um container. Você pode pensar em uma imagem como um snapshot de uma máquina e um container como sendo a instância em execução real da máquina. Dockerfiles sempre possuem o formato:

```
INSTRUCTION arguments
```

Assim, em nosso exemplo hello-world, podemos dar uma olhada na raiz do repositório no GitHub que contém o Dockerfile. A imagem é criada a partir de uma outra imagem chamada “[scratch](https://registry.hub.docker.com/u/library/scratch/)” (todos os Dockerfiles começam com a instrução FROM), em seguida, copia o [arquivo hello](https://github.com/docker-library/hello-world/blob/master/hello) para a raiz do sistema, e, finalmente, executa o_hello_. Você também pode encontrar o [conteúdo do arquivo hello aqui](https://github.com/docker-library/hello-world/blob/master/hello.asm), que contém a saída que acabamos de ver no nosso terminal.

### Docker Pull: Baixando uma imagem Ubuntu

Agora que sabemos que a nossa instalação do Docker está funcionando, vamos começar a brincar com ele! Nosso próximo passo é obter uma imagem Ubuntu. Para encontrar imagens nós podemos ir diretamente para o [site do Docker Hub](https://hub.docker.com/) ou simplesmente executar no terminal:

```bash,linenums=true
$ docker search ubuntu
```

Isso vai mostrar uma lista de todas as imagens que contenham Ubuntu em seu nome. O output no meu terminal após executar o comando:

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*VyLXJYEaN9BQwQDRWsMZiw.png" alt="terminal docker search"/>
</p>

O resultado é ordenada pelo número de estrelas em cada repositório das imagens (assim como no GitHub, você pode acessar o Docker Hub e marcar imagens como favoritas). Note que há uma coluna “Official” e outra “Automated”.

*   **Imagens oficiais** são mantidas pelo projeto docker-library e devem ser aceitas pela equipe da Docker. Isso significa que essas imagens aderem a algumas guidelines encontradas [aqui](https://docs.docker.com/docker-hub/official_repos/), como por exemplo, estarem em um repositório git e esse repositório tendo pelo menos permissão de leitura para que os usuários possam verificar o seu conteúdo. Você pode confiar que essas imagens irão funcionar corretamente com o Docker. Além disso, ao contrário de outras imagens em que você precisa fazer referência a elas durante o download usando USERNAME/IMAGE_NAME, estas imagens podem simplesmente ser referidas em comandos por IMAGE_NAME (como o Ubuntu). Todos os Dockerfiles dessas imagens oficiais podem ser encontradas [nesta organização](https://github.com/docker-library).
*   A coluna "Automated" refere-se a imagens com "[Automated Build](https://docs.docker.com/userguide/dockerrepos/#automated-builds)". Significa simplesmente que a imagem está sendo construída a partir de um Dockerfile dentro de um repositório GitHub ou BitBucket, e essa imagem é atualizada automaticamente quando alterações são feitas ao repositório.

Vamos baixar a imagem oficial Ubuntu:

```bash,linenums=true
$ docker pull ubuntu
```

O comando `$ docker pull IMAGE_NAME` é a maneira de baixar explicitamente uma imagem, mas isso também é feito se você utilizar o comando `$ docker run IMAGE_NAME`, e o Docker não conseguir encontrar a imagem a qual você está se referindo como já existente no seu sistema.

### Docker Run: Executando nossa imagem Ubuntu e acessando um container

Nós baixamos a imagem Ubuntu (nossa "planta" ☺). Agora vamos começar um novo container com base em nossa imagem, e ao mesmo tempo passar um comando para ele:

```bash,linenums=true
$ docker run ubuntu /bin/echo ‘Hello world’
```

Isso deve mostrar em seu terminal a mensagem “Hello World”. Bom, é legal que nós iniciamos um container executando uma instância completamente isolada do Ubuntu e executamos um comando, mas isso não é muito útil.

Então, agora, vamos executar um novo container com Ubuntu e conectar nele:

```bash,linenums=true
$ docker run -i -t ubuntu
```
> Nota: O comando “run” é enorme (veja o `$ docker help run`) e nós iremos nos aprofundar nele no próximo post

O parâmetro -t atribui um "pseudo-tty" ou terminal dentro de nosso container e o parâmetro -i nos permite fazer uma conexão interativa, pegando o stdin do container. Caso tenha funcionado corretamente, você deve estar conectado a um terminal dentro do recipiente mostrando algo como:

```bash,linenums=true
$ root@c9989236296d:/# 
```

Execute "ls -ls" e veja que seu comando está sendo executado no root de um sistema Ubuntu. ☺

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*rlznCcFV8BcJXKDxIf35Lw.png" alt="terminal ubuntu"/>
</p>

Eu acho que é bom parar por um minuto e pensar sobre o que acabamos de fazer. Esta é apenas uma das partes impressionantes de containers. Nós acabamos de baixar e iniciar um container rodando Ubuntu. Isso aconteceu (dependendo da sua conexão com a internet) em 5 minutos? Compare isso com o download de uma imagem de Ubuntu, iniciando e instalando essa imagem em uma nova VM. Isso provavelmente iria levar em torno de 15–30min? E, em seguida, a criação de novas máquinas virtuais, parar, reiniciar, quanto tempo isso levaria? Quando você adiciona todas essas ações, o tempo que você pode economizar utilizando containers é enorme!

### Docker Commit: Instalando node, npm, express e fazendo “commit” de nossas alterações

Ok, agora que estamos dentro de um container Ubuntu em execução, vamos instalar as ferramentas que precisamos para executar uma aplicação node (lembre-se que você só precisa executar a parte depois de `$ root:`):

```bash,linenums=true
$ root: apt-get update  
$ root: apt-get install nodejs  
$ root: apt-get install nodejs-legacy
```
> _Nota: Nós precisamos instalar `nodejs-legacy` para executar o módulo express-generator_

Se executarmos `node -v` devemos ver a mensagem:

```bash,linenums=true
$ root: node -v  
v0.10.25
```

Com node instalado, podemos seguir em frente e instalar o [módulo express-generator](http://expressjs.com/starter/generator.html) do npm:

```bash,linenums=true
$ root: npm install -g express-generator
```

Agora temos nosso container com tudo que precisamos instalado! Vamos sair dele:

```bash,linenums=true
$ root: exit
```

Ao fazermos isso, o Docker irá parar a execução dele. Podemos utilizar o comando `$ docker ps` para listar os containers então execute:

```bash,linenums=true
$ docker ps -a
```

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/2000/1*l-NGj6PaW9F15wsiOo7OzA.png" alt="terminal containers"/>
</p>

O comando `$ docker ps` por padrão apenas mostra os containers em execução, então passamos o atributo -a para que possamos ver o nosso container Ubuntu que acabamos de sair.

Agora podemos usar esse container para criar uma nova imagem que outras pessoas possam usar. Fazemos isso usando o comando `commit`:

```bash,linenums=true
$ docker commit -a "Your Name &lt;youremail@email.com&gt;" -m "node and express" CONTAINER_ID node-express:0.1
```
> _Nota: Altere o conteúdo do parâmetro -a, e o_ CONTAINER_ID_com o ID de seu container mostrado no output de_ `$ docker ps -a`._Você pode usar apenas os primeiros 3/4 caracteres do ID_. ☺

O comando "commit" recebe alguns parâmetros. A parâmetro -a define o autor, você pode definir uma mensagem de commit usando o parâmetro -m, e, finalmente, a nossa referência ao container ID e o nome da imagem que estamos criando, neste caso `node-express`. Nós também definimos uma tag para a nossa imagem, adicionando o `: 0.1` após o nome da imagem. Se executarmos agora:

```bash,linenums=true
$ docker images
```

Nós devemos ver:

<p align="center">
  <img src="https://d262ilb51hltx0.cloudfront.net/max/1600/1*1pF330DlUkoTsUrae8dRKg.png" alt="terminal images"/>
</p>

Parabéns! Você acabou de criar sua primeira imagem no Docker!

Agora vamos adicionar uma nova tag na nossa recém-criada imagem. Execute:

```bash,linenums=true
$ docker tag node-express:0.1 node-express:latest
```

É uma boa prática taggear imagens com uma versão específica para que outras pessoas possam saber exatamente qual a imagem que eles estão executando. Adicionar a tag `latest` ajuda com que outras pessoas possam simplesmente se referir a sua imagem pelo nome dela, sem a versão, para baixá-la ou executá-la (node-express no nosso caso). O Docker irá pegar automaticamente a versão com a tag `latest`. Se você executar `$ docker images` novamente, verá que há duas linhas com nossa imagem, mas ambas têm o mesmo ID, o que significa que elas não estão ocupando espaço extra no nosso HD. ☺

Agora podemos começar quantos containers quisermos com a nossa imagem! Vamos remover o nosso container anterior:

```bash,linenums=true
$ docker ps -a   
$ docker rm YOUR_CONTAINER_ID
```
> _Nota: Lembre-se que você pode utilizar os 3–4 primeiros caracteres do ID_

E vamos executar um container com base na nossa nova imagem, conectar nele usando os parâmetros -t e -i, e expor a porta 8080 do host (VirtualBox) como a porta 3000 do container (VM):

```bash,linenums=true
$ docker run -i -t -p 8080:3000 node-express
```

Agora que estamos conectados, vamos usar o express-generator que instalamos anteriormente para criarmos uma nova aplicação Node.js:

```bash,linenums=true
$ root: express mynodeapp
```

Seguindo as instruções do terminal, entre na pasta do novo app, instale as dependências e inicie a aplicação:

```bash,linenums=true
$ root: cd mynodeapp  
$ root: npm install  
$ root: npm start
```

Agora, temos uma aplicação Node.js sendo executada dentro de um container, e expondo a porta 3000\. Para ver a nossa aplicação, precisamos encontrar o IP da VM Boot2Docker, para isso abra outro terminal e execute:

```bash,linenums=true
$ boot2docker ip  
192.168.59.103
```

E lembre-se que nós expomos a porta 8080 do nosso container para acessar a porta 3000\. Então abra um browser e digite:

```bash,linenums=true
192.168.59.103:8080
```

Ta-ra!

<figure name="bc16" id="bc16" class="graf--figure"><div class="aspectRatioPlaceholder is-locked" style="max-width: 700px; max-height: 464px;">![](https://d262ilb51hltx0.cloudfront.net/max/1600/1*NHmZcrvxfAl3jQ0Ow1N6lw.png)</div></figure>

Agora, você pode começar a se perguntar: isso é muito trabalho para executar uma aplicação! Eu já tenho o meu ambiente de desenvolvimento, eu poderia ter feito tudo isso em 30 segundos! Bem, isso é verdade, mas neste tutorial nós estamos executando um aplicativo super simples que não possui muitas dependências. Quando você está executando um projeto real que tem muito mais dependências, você pode necessitar de um ambiente de desenvolvimento com diferentes pacotes, Python, Redis, MongoDB, Postgres, Node.js ou io.js, etc. Existem diversas coisas envolvidas, que podem fazer **a execução de uma aplicação no seu computador não funcionar corretamente em outra máquina** (ou em QA / Teste / Produção), e isso é exatamente a principal razão pela qual Docker se tornou tão popular. Voltando para a introdução deste tutorial, **fornecendo uma unidade fundamental** (o nosso container real/ peça de Lego) que pode ser executado independente de hardware, e também iniciado, movido, compartilhado, Docker absolutamente mudou a forma como podemos desenvolver, testar e compartilhar aplicações.
  
### Docker Push: Fazendo “push” do nosso container para que outras pessoas possam utiliza-lo

Ok, agora vamos compartilhar nossa “incrível” imagem Ubuntu com node, npm, e express-generator instalados para que outras pessoas também possam usá-la. Saia da nossa aplicação Node em execução e do container:

```bash,linenums=true
# Ctrl+C to stop our node app  
$ root: exit
```

Acesse o Docker Hub para criar uma conta grátis: [http://hub.docker.com](http://hub.docker.com)

Após isso, volte para seu terminal e faça o login:

```bash,linenums=true
$ docker login
```

Agora que estamos logados no cli, podemos fazer o "push" de nossa imagem para o Docker Hub. Primeiro, vamos renomea-la adicionando nosso username, então assim como adicionamos uma tag antes:

```bash,linenums=true
$ docker tag node-express your_docker_hub_username/node-express  
$ docker rmi node-express  
$ docker push your_docker_hub_username/node-express
```

Feito! Agora qualquer um com Docker instalado pode executar:

```bash,linenums=true
$ docker pull your_docker_hub_username/node-express
```

E ter exatamente o mesmo ambiente com Ubuntu, Node.js, npm e o pacote express-generator que usamos anteriormente.

### Próximo post: Adicionando Docker a uma aplicação existente, executando e conectando containers

Esta é uma longa introdução, e ainda há muito para cobrirmos, mas você deve estar equipado com uma compreensão básica do que é o Docker, como usar suas funcionalidades básicas, e pronto para aprender mais.

No próximo tutorial, eu vou cobrir como adicionar um Dockerfile a uma aplicação existente, aprofundar nosso conhecimento do comando `$ docker run`, montar o diretório de uma aplicação dentro de um container, e fazer uma conexão com outro container executando uma instância MongoDB. ☺

</div></div></section></section>