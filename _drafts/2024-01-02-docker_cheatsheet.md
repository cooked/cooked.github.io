---
layout: single
title: Docker cheatsheet

toc: true
toc_label: "Contents"
toc_icon: "none"

tag: [Docker, Software]

header:
    teaser: /assets/img/apple/terminal.png
---

All the Docker commands I always forget.

## Build 

```bash
docker build -t <tagname> .
```

## Delete all unused

```bash
docker system prune -a
```

## Show size of image layers

```bash
# list images
# pick one and show layers size
docker images -a
docker history <Image ID>
```

## References

- [The Ultimate Docker Cheat Sheet](https://dockerlabs.collabnix.com/docker/cheatsheet/)
