---
{"dg-publish":true,"lang":"en","dg-created":"2023-09-23T19:00:00","dg-updated":"2024-02-13T11:11:00","tags":["docker","buildah","multiarch","sre"],"permalink":"/en/sre-and-friends/building-multiarch-images-with-buildah/","dgPassFrontmatter":true,"created":"2023-09-23T19:00:00","updated":"2024-02-13T11:11:00"}
---

[[en/Home\|Home]] >> [[en/SRE and Friends/index\|SRE and Friends]] >> Building multiarch images with Buildah [[pt/SRE e Amigos/Construindo imagens multiplataforma com o Buildah\|🇧🇷]]

![buildah_logo.png|Logo from Buildah (a dog wearing a constructor hat), an icon representing the OCI (Open Container Initiative) standard and an icon representing a built container image](/img/user/assets/buildah_logo.png)
# Building multiarch images with Buildah

With the growth and popularization of platforms beyond amd64, especially the ARM platform, it becomes a concern for those maintaining container images to build them tailored for these platforms as well. 

In this article, I will show you how to build these images using [Buildah](https://buildah.io/).

Buildah is an open-source project led by RedHat (but receives contributions from other companies) that aims to simplify the construction of container images that adhere to and are compatible with the [OCI (Open Container Initiative)](https://opencontainers.org/) standard.

So let's get to the steps.
## Prerequisites

- Installation of the Buildah package

- Installation of the `qemu-user-static` package (Ubuntu/RedHat) or `qemu-arch-extra` package (Arch Linux);

```shell
sudo dnf install -y buildah qemu-user-static #Fedora and derivatives

sudo apt-get install -y buildah qemu-user-static #Ubuntu/Debian and derivatives

sudo pacman -Sy buildah qemu-arch-extra #Arch Linux and derivatives
```
## Building the images

The workflow you need to follow to build the images is as follows:

1. Create the manifest

```bash
export MANIFEST="multiarch-image" # manifest name, can be any name
export IMAGE="registry/repository/image:tag" # image name, e.g., docker.io/ozorest/example:latest
buildah manifest create "$MANIFEST"
```

2. Run the build for each architecture

In the directory with your Dockerfile, you can run the following loop for each architecture:

```bash
for arch in amd64 arm64; do
buildah build --arch $arch --tag "$IMAGE" --manifest "$MANIFEST" .
# buildah build --arch $arch --tag "$IMAGE" --manifest "$MANIFEST" -f path_to_Dockerfile, in case the Dockerfile is not in the current directory
done
```

3. Push the manifest to the registry

```bash
buildah manifest push --all "$MANIFEST" "docker://$IMAGE"
```

And that's it, folks! Stay tuned for more content!