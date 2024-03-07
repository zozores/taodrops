---
{"dg-publish":true,"lang":"pt","dg-created":"2024-02-18T10:53:00","dg-updated":"2024-02-18T17:50:00","tags":["paas","self-hosted","opensource","foss","one-click"],"permalink":"/pubnotes/pt/cap-rover-um-paa-s-para-chamar-de-seu/","dgPassFrontmatter":true,"created":"2024-02-18T10:53:00","updated":"2024-02-18T17:50:00"}
---

[[pubnotes/pt/Home\|Home]] >> CapRover - Um PaaS para chamar de seu [[pubnotes/en/CapRover - PaaS to call your own\|🇬🇧]]

![caprover_logo.png|Imagem mostrando o logotipo do CapRover, que parece uma chama de fogo](/img/user/assets/caprover_logo.png)

Se tem um projeto código aberto que eu não vivo mais sem ele e está sendo uma mão na roda para mim quando preciso fazer deploy rápido de aplicações containeirizadas, é o [CapRover](https://caprover.com)

O CapRover é um PaaS (Plataforma como Código), que você pode instalar em qualquer infra pública que tenha o Docker instalado, ou seja, se você tem uma máquina na Digital Ocean, na AWS, na OCI (é o que eu estou usando, tenho uma máquina usando o Free Tier lá e está me atendendo muito bem até o momento) ou em qualquer outra cloud, você consegue facilmente instalá-lo e ter um PaaS "um clique" para chamar de seu.

![magic.gif](/img/user/assets/magic.gif)

## Pré-requisitos

Já dei um spoiler de alguns dos pré-requisitos para instalação, mas vamos listar todos eles para ficar mais claro do que você precisa:

- **Um servidor com IP público**, porque na instalação padrão está tudo preparado para rodar em uma infra pública, é possível instalar em uma infra local, mas [alguns passos a mais]((https://caprover.com/docs/run-locally.html) são necessários.
- **Nome de Domínio**, pois durante a instalação o CapRover irá solicitar a configuração de uma entrada "wildcard" no DNS apontando para este IP público do CapRover.
- Uma **entrada A "wildcard" no DNS** do domínio, como no exemplo abaixo![entrada_dns.png|Imagem mostrando a configuração de um wildcard no DNS da CloudFlare](/img/user/assets/entrada_dns.png)
- **Docker** instalado neste servidor.
- Seguintes **portas abertas no firewall** da máquina, já que alguns provedores podem restringir a abertura de algumas dessas portas por padrão nas máquinas
	- 80,443,3000,996,7946,4789,2377/tcp
	- 7946,4789,2377/udp

## Instalação

Com todos pré-requisitos prontos, podemos partir para instalação do CapRover, que começa você acessando a sua máquina via SSH e executando um container docker que subirá todos os componentes necessários para o CapRover funcionar.

```shell
docker run -p 80:80 -p 443:443 -p 3000:3000 -e ACCEPTED_TERMS=true -v /var/run/docker.sock:/var/run/docker.sock -v /captain:/captain caprover/caprover
```

Após a subida dos componentes é chegada a hora de instalar o CLI do CapRover na sua máquina local (laptop ou desktop), pois vai ser através dele que você finalizará a instalação do CapRover no seu servidor, a instalação do CLI pode ser feita usando o NPM, porém em algumas distros do Linux (por exemplo, o Arch Linux), é possível instalar através do gerenciador de pacotes.

```shell
npm -g install caprover
```

E após a instalação do CLI, basta executar o comando abaixo para finalizar a configuração do servidor:

```shell
caprover serversetup
```

Em algum momento da configuração que se dará de forma interativa, ou seja, ele vai te perguntando como você vai querer configurar o CapRover, ele irá perguntar sobre aquele o domínio e aquele entrada wildcard que você configurou no DNS.

![caprover_serversetup.png|Imagem mostrando cada uma das opções que CapRover disponibiliza no modo interativo, ele pergunta sobre o endereço de IP do servidor, o domínio e DNS configurado, o email para habilitar o HTTPS e o nome da máquina do CapRover](/img/user/assets/caprover_serversetup.png)

Finalizada configuração é só acessar a URL https://captain.dominioescolhido para acessar a console do CapRover.

![caprover_login.png|Imagem mostrando a tela de login do CapRover](/img/user/assets/caprover_login.png)

Para mais informações sobre o processo de instalação, vocês podem acessar a [documentação do produto](https://caprover.com/docs/get-started.html) ou assistir a este [vídeo tutorial](https://www.youtube.com/watch?v=VPHEXPfsvyQ)criado pelo desenvolvedor da ferramenta.

## Deploy de Aplicações

Uma das coisas mais bacanas é a flexibilidade que você tem para fazer o deployment das aplicações, o CapRover suporta 7 métodos diferentes de fazer o deployment da sua aplicação:

1. Via CLI usando o comando `caprover deploy` no diretório raiz do projeto que você que fazer o deploy (forma recomendada para uso em um CI/CD)
2. Via a interface web fazendo o upload de um arquivo tar contendo o projeto para deploy
3. Via interface web configurando um webhook para deploy de forma automática a cada push em um repositório no Github, Gitlab, Bitbucket, etc (outra forma recomendada de uso em um CI/CD)
4. Via interface web usando um Dockerfile
5. Via interface web usando um arquivo *captain-definition*, que eu já explico do que se trata (==os métodos de 1 a 3 também precisam deste arquivo na raiz do projeto==)
6. Via interface web usando uma imagem já existente em um Docker Registry (caso você não configure outro, o padrão é usar o [Docker Hub](https://hub.docker.com))

![](https://i.imgur.com/SyOf7RY.png)
![](https://i.imgur.com/7YMBunc.png)

Ué, mas você não falou 7 métodos!?

Calma, coração aflito, eu deixei o mais legal para o final 😁

O último método é através de uma loja de aplicativos "um clique" (eu já tinha dado esse spoiler, eu sei), ou seja, com algumas configurações e um clique você pode instalar um Wordpress, por exemplo.

![](https://i.imgur.com/WOS7t0u.png)

![](https://i.imgur.com/U5HcE4j.png)

O CapRover já instala um [repositório de aplicativos](https://github.com/caprover/one-click-apps) "um clique" padrão, que já vem com boas opções de aplicativos pré-configurados, para conferir a lista é só entrar neste [caminho do repositório](https://github.com/caprover/one-click-apps/tree/master/public/v4/apps), nesse caminho também já dá para ter uma ideia de como são escritos os modelos e que como vocês podem ver são arquivos YAML com algumas variáveis especiais.

```yaml
captainVersion: 4
services:
    $$cap_appname:
        image: linuxserver/filezilla:$$cap_fz_version
        environment:
            TZ: $$cap_tz
            PUID: '1000'
            PGID: '1000'
        restart: unless-stopped
        volumes:
            - $$cap_appname-config:/config
        caproverExtra:
            containerHttpPort: '3000'
caproverOneClickApp:
    variables:
        - id: $$cap_fz_version
          label: Filezilla Version
          defaultValue: 3.51.0-r1-ls6
          description: Check out their Docker page for the valid tags https://hub.docker.com/r/linuxserver/filezilla/tags
          validRegex: /^([^\s^\/])+$/
        - id: $$cap_tz
          label: Time Zone
          defaultValue: Asia/Kolkata
          description: Get yours from https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
          validRegex: /.{1,}/
    instructions:
        start: >-
            FIleZilla Client is a fast and reliable cross-platform FTP, FTPS and SFTP client with lots of useful features and an intuitive graphical user interface.
        end: >-
            Filezilla is deployed and available as http://$$cap_appname.$$cap_root_domain.
            IMPORTANT!! You need to enable websocket support.
            The default username/password is abc/abc.
            By default the user/pass is abc/abc, if you change your password or want to login manually to the GUI session for any reason use the following link: http://$$cap_appname.$$cap_root_domain/?login=true
    displayName: Filezilla
    isOfficial: true
    description: FileZilla is a free and open-source, cross-platform FTP application
    documentation: Taken from https://hub.docker.com/r/linuxserver/filezilla.
```

Não sei se vocês perceberam, tudo que está dentro do atributo `services` é exatamente como estaria em um arquivo docker-compose, então você pode portar facilmente o seu docker-compose para o modelo do CapRover.

E o melhor é que você pode criar um fork desse repositório e criar o seu próprio repositório, a sua própria lojinha de aplicativos, inclusive no [README do repo](https://github.com/caprover/one-click-apps/blob/master/README.md) já tem instruções de como fazer isso.

Você também pode procurar por *[caprover-one-click-apps](https://github.com/search?q=caprover-one-click-apps&type=repositories)* no Github, que é bem provável que você encontre outros repositórios de terceiros com mais opções de aplicativos.

> [!TIP] Rodando o CapRover em arquiteturas ARM
> Muitas aplicações ainda não tem imagens prontas para arquitetura ARM, quando você fizer o deploy de uma aplicação que não tem uma imagem para esta plataforma, você verá este erro nos logs da aplicação na interface Web:
> ![](https://i.imgur.com/CaF1bqp.png)
> Como lidar com isso, você pode ver em um outro artigo que eu escrevi [[pubnotes/pt/Construindo imagens multiplataforma com o Buildah\|Construindo imagens multiplataforma com o Buildah]]
## Arquivo captain-definition

Este é um formato de arquivo do CapRover que você usa para definir como a sua aplicação, o seu projeto irá subir no CapRover, qual imagem ele usará, se vai usar um Dockerfile de referência ou você pode escrever um Dockerfile direto dentro deste arquivo, que é um arquivo JSON simples, por exemplo:

```json
{
"schemaVersion": 2,
"dockerfileLines": [
	"FROM node:8.7.0-alpine",
	"RUN mkdir -p /usr/src/app",
	"WORKDIR /usr/src/app",
	"COPY ./package.json /usr/src/app/",
	"RUN npm install && npm cache clean --force",
	"COPY ./ /usr/src/app",
	"ENV NODE_ENV production",
	"ENV PORT 80",
	"EXPOSE 80",
	"CMD [ \"npm\", \"start\" ]"
	]
}
```

Usando um Dockerfile:

```json
{ 
	"schemaVersion": 2, 
	"dockerfilePath": "./Dockerfile" 
}
```

Usando uma imagem de um repositório Docker:

```json
{ 
	"schemaVersion": 2, 
	"imageName": "nginxdemos/hello" 
}
```

E é isso pessoal, eu fico por aqui e fiquem ligados nas próximas notas!