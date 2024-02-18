---
{"dg-publish":true,"lang":"en","dg-created":"2024-02-18T10:53:00","dg-updated":"2024-02-18T17:50:00","tags":["paas","self-hosted","opensource","foss","one-click"],"permalink":"/en/brave-open-source/cap-rover-paa-s-to-call-your-own/","dgPassFrontmatter":true,"created":"2024-02-18T10:53:00","updated":"2024-02-18T17:50:00"}
---

[[pt/Home\|Home]] >> [[en/Brave Open Source/index\|Brave Open Source]] >> CapRover - PaaS to call your own [[pt/Admiravel Codigo Aberto/CapRover - Um PaaS para chamar de seu\|🇧🇷]]

![caprover_logo.png|Image showing the CapRover logo, which looks like a flame of fire](/img/user/assets/caprover_logo.png)

If there's one open-source project that I can't live without anymore and that's been a godsend for me when I need to quickly deploy containerized applications, it's [CapRover](https://caprover.com).

CapRover is a PaaS (Platform as Code), which you can install on any public infrastructure that has Docker installed, i.e. if you have a machine on Digital Ocean, AWS, OCI (that's what I'm using, I have a machine using Free Tier there and it's serving me very well so far) or any other cloud, you can easily install it and have a "one-click" PaaS to call your own.

![magic.gif](/img/user/assets/magic.gif)

## Prerequisites

I've already spoiled some of the prerequisites for installation, but let's list them all to make it clearer what you need:

- **A server with a public IP**, because in the standard installation everything is set up to run on a public infrastructure, it is possible to install on a local infrastructure, but [a few more steps]((https://caprover.com/docs/run-locally.html) are required.
- Domain Name**, because during installation CapRover will ask you to configure a "wildcard" DNS entry pointing to this CapRover public IP.
- A **wildcard entry in the domain's DNS**, as in the example below ![entrada_dns.png|Image showing the configuration of a wildcard in CloudFlare DNS](/img/user/assets/entrada_dns.png)
- **Docker** installed on this server.
- The following **ports open in the machine's firewall**, as some providers may restrict the opening of some of these ports by default on their machines
	- 80,443,3000,996,7946,4789,2377/tcp
	- 7946,4789,2377/udp

## Installation

With all the prerequisites ready, we can start installing CapRover, which begins with you accessing your machine via SSH and running a docker container that will up all the components needed for CapRover to work.

```shell
docker run -p 80:80 -p 443:443 -p 3000:3000 -e ACCEPTED_TERMS=true -v /var/run/docker.sock:/var/run/docker.sock -v /captain:/captain caprover/caprover
```

Once the components have been up, it's time to install the CapRover CLI on your local machine (laptop or desktop), as this is how you will finalize the installation of CapRover on your server. The CLI can be installed using NPM, but in some Linux distros (e.g. Arch Linux), it can be installed using the package manager.

```shell
npm -g install caprover
```

And after installing the CLI, just run the command below to finish configuring the server:

```shell
caprover serversetup
```

At some point during the configuration, which will take place interactively, in other words, it will ask you how you want to configure CapRover, it will ask you about that domain and that wildcard entry that you have configured in DNS.

![caprover_serversetup.png|Image showing each of the options that CapRover provides in interactive mode, it asks for the IP address of the server, the domain and DNS configured, the email to enable HTTPS and the name of the CapRover machine](/img/user/assets/caprover_serversetup.png)

Once the configuration is complete, simply access the URL https://captain.chosendomain to access the CapRover web interface.

![caprover_login.png|Image showing the CapRover login screen](/img/user/assets/caprover_login.png)

For more information about the installation process, you can access the [product documentation](https://caprover.com/docs/get-started.html) or watch this [video tutorial](https://www.youtube.com/watch?v=VPHEXPfsvyQ) created by the tool's developer.

## Application Deployment

One of the coolest things about CapRover is the flexibility you have to deploy applications. CapRover supports 7 different methods of deploying your application:

1. Via CLI using the `caprover deploy` command in the root directory of the project you want to deploy (recommended for use on a CI/CD).
2. Via the web interface by uploading a tar file containing the project for deployment
3. Via the web interface by configuring a webhook to deploy automatically with each push to a repository on Github, Gitlab, Bitbucket, etc. (another recommended way of using a CI/CD)
4. Via web interface using a Dockerfile
5. Via web interface using a *captain-definition* file, which I'll explain in a moment (==methods 1 to 3 also need this file in the root of the project==)
6. Via the web interface using an existing image in a Docker Registry (if you haven't set one up, the default is to use [Docker Hub](https://hub.docker.com))

![](https://i.imgur.com/SyOf7RY.png)
![](https://i.imgur.com/7YMBunc.png)

But didn't you say 7 methods?

Calm down, I've left the coolest for last 😁

The last method is via a "one-click" app store (I've already given away this spoiler, I know), meaning that with a few settings and one click you can install Wordpress, for example.

![](https://i.imgur.com/WOS7t0u.png)

![](https://i.imgur.com/U5HcE4j.png)

CapRover already installs a standard "one-click" [application repository](https://github.com/caprover/one-click-apps), which already comes with a good selection of pre-configured applications. To check out the list, just go to this [repository path](https://github.com/caprover/one-click-apps/tree/master/public/v4/apps), and you'll also get an idea of how the templates are written, which as you can see are YAML files with some special variables.

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

I don't know if you noticed, everything inside the `services` attribute is exactly as it would be in a docker-compose file, so you can easily port your docker-compose to the CapRover model.

And the best thing is that you can fork this repository and create your own repository, your own app store, even in the [README of the repo](https://github.com/caprover/one-click-apps/blob/master/README.md) there are instructions on how to do this.

You can also search for *[caprover-one-click-apps](https://github.com/search?q=caprover-one-click-apps&type=repositories)* on Github, which is likely to find other third-party repositories with more application options.

> [!TIP] Running CapRover on ARM architectures
> Many applications do not yet have images ready for ARM architecture. When you deploy an application that does not have an image for this platform, you will see this error in the application logs in the Web interface:
> ![](https://i.imgur.com/CaF1bqp.png)
> How to deal with this, you can see in another article I wrote [[en/SRE and Friends/Building multiarch images with Buildah\|Building multiarch images with Buildah]]
## captain-definition File

This is a CapRover file format that you use to define how your application, your project will upload to CapRover, which image it will use, whether it will use a reference Dockerfile or you can write a direct Dockerfile inside this file, which is a simple JSON file, for example:

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

Using a Dockerfile

```json
{ 
	"schemaVersion": 2, 
	"dockerfilePath": "./Dockerfile" 
}
```

Using an image from a Docker repository

```json
{ 
	"schemaVersion": 2, 
	"imageName": "nginxdemos/hello" 
}
```

And that's it folks, I'm done here and stay tuned for the next notes!