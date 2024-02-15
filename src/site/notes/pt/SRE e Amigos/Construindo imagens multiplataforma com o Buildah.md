---
{"dg-publish":true,"lang":"pt","dg-created":"2023-09-22T19:00:00","dg-updated":"2024-02-13T11:07:00","permalink":"/pt/sre-e-amigos/construindo-imagens-multiplataforma-com-o-buildah/","dgPassFrontmatter":true,"created":"2023-09-22T19:00:00","updated":"2024-02-13T11:07:00"}
---

[[pt/Home\|Home]] >> [[pt/SRE e Amigos/index\|SRE e Amigos]] >> Construindo imagens multiplataforma com o Buildah

🇺🇸 🇬🇧 [[en/SRE and Friends/Building multiarch images with Buildah\|English version]]

![buildah_logo.png|Logotipo do Buildah (um cachorro usando um chapéu de construtor), um ícone representando o padrão OCI (Open Container Initiative) e um ícone representando uma imagem de contêiner construída](/img/user/assets/buildah_logo.png)
# Construindo imagens multiplataforma com o Buildah

Com o crescimento e a popularização de outras plataformas além da amd64, principalmente a plataforma ARM, passa a ser uma preocupação de quem mantém imagens de containers, a construção destas voltados para estas plataformas também.

E neste artigo eu mostrarei como construir essas imagens usando o [Buildah](https://buildah.io/).
 
O Buildah é um projeto open-source capitaneado pela RedHat (mas que recebe contribuições de outras empresas) que visa facilitar a construção de imagens de containers que respeitem e sejam compatíveis com o padrão da [OCI (Open Container Iniciative)](https://opencontainers.org/)

Então vamos aos passos.

## Pré-requisitos

- Instalação do pacote do Buildah

- Instalação do pacote do `qemu-user-static` (Ubuntu/RedHat) ou `qemu-arch-extra` (Arch Linux);

```shell
sudo dnf install -y buildah qemu-user-static #Fedora e derivados

sudo apt-get install -y buildah qemu-user-static #Ubuntu/Debian e derivados

sudo pacman -Sy buildah qemu-arch-extra #Arch Linux e derivados
```

## Construindo as imagens 

O fluxo que você precisa seguir para construir as imagens é o seguinte: 

1. Criar o manifesto

```bash
export MANIFEST="imagem-multiarch" # nome do manifesto, pode ser qualquer nome
export IMAGE="registry/repositorio/imagem:tag" # nome da imagem, ex: docker.io/ozorest/example:latest
buildah manifest create "$MANIFEST"
``` 

2. Rodar o build para cada uma das arquiteturas

No diretório do seu Dockerfile, você pode rodar o loop abaixo para cada arquitetura: 

```bash
for arch in amd64 arm64; do
buildah build --arch $arch --tag "$IMAGE" --manifest "$MANIFEST" .
# buildah build --arch $arch --tag "$IMAGE" --manifest "$MANIFEST" -f caminho_do_Dockerfile, caso o Dockerfile não esteja no diretório atual
done
``` 

3. Enviar o manifesto para o registry

```bash
buildah manifest push --all "$MANIFEST" "docker://$IMAGE"
``` 

E isso é tudo, pessoal! E fique ligado para mais conteúdos!