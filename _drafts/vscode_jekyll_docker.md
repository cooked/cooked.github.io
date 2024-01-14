---
layout: single
title: Jekyll container in VSCode

tags: [Jekyll, GitHub Pages, Docker, VSCode]
# categories: software, hardware, embedded 
category: software

#header:
#    teaser: /assets/img/jvscd.png
---

This post describes how develop static websites with Jekyll and VSCode, inside a Docker container.

## TL;DR

I've put toghether a VSCode project to develop static websites with [Jekyll](https://jupyter.org/) running inside a [Docker](https://www.docker.com/) container. It will take care of the mundane configuration tasks for you. Check it out on [GitHub](https://github.com/cooked/jupyter-devbox/tree/custom).

## Setup

First things first, make sure [Docker](https://www.docker.com/) is installed on your system; on Linux this can be done with the following command:

``` bash
sudo apt-get install docker-ce
```

Next, go ahead download and install Visual Studio Code from the [official page](https://code.visualstudio.com/download) and once make sure to add the [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension ,which lets you use a Docker container as a full-featured development environment.

### Dev Containers

As it does for other IDE features, VScode offers a mechanism to have container-related settings configured and persisted in a .json file. A Dev Containers-aware folder/workspace only needs a [.devcontainer](){: .btn .btn--primary .btn--small} folder with two files in it:

- [devcontainer.json](https://containers.dev/implementors/json_reference/){: .btn .btn--info .btn--small} which contains any needed metadata and settings required to configurate a development container for a given well-defined tool and runtime stack
- [Dockerfile](https://docs.docker.com/engine/reference/builder/){: .btn .btn--info .btn--small} a text document that contains all the commands a user could call on the command line to assemble a Docker image
  
 Basically, the Dev Containers take an existing Dockerfile, fully compatible with standard Docker, and add some metadata to better integrate it in a typical VSCode development workflow e.g., launching Docker in the background, hiding away some mundane tasks, automate some others and most important let you focus on the development that really matters.
 {: .notice}

As most of the times happens these days we can avoid writing our Dev Container from scratch and get inspired by some of the existing ones. In the particular case of Jupyter, the [jupyter-devbox](https://github.com/jakoch/jupyter-devbox) project is an excellent starting point.

## Jupyter Devbox

The [jupyter-devbox](https://github.com/jakoch/jupyter-devbox) is a project that wraps most of what described above in a ready-to-use folder that comes with a Dockerfile tailored to machine learning and computer vision development.  

Instead of cloning the original repo I worked on my own and made some changes, available on [GitHub](https://github.com/cooked/jupyter-devbox).

## Customizations

Things I've changed from the original jupyter-devbox are:

- removed machine learning and computer vision packages
- added *pandoc* to export notebooks to non-HTML formats
- added *texlive* to export notebooks to PDF format
- added Python packages to work with *.xlsx* and [InfluxDB](https://www.influxdata.com/)

Check them out in the new [Dockerfile](https://github.com/cooked/jupyter-devbox/blob/custom/.devcontainer/Dockerfile)

### Jupyter notebook export

On top of the customizations described above, I've added [.vscode/tasks.json](https://github.com/cooked/jupyter-devbox/blob/custom/.vscode/tasks.json){: .btn .btn--primary .btn--small} to leverage the automation exposed by the [VSCode Tasks](https://code.visualstudio.com/docs/editor/tasks) framework.  

In my case I wanted to have quick access to converting and exporting the current notebook, so I've added few custom tasks like so:

``` json
"tasks": [
    {
        "label": "New Task",
        "type": "shell",
        "group": "build",
        "command": "jupyter-nbconvert",
        "args": [
            "--to", "pdf",
            "--TemplateExporter.exclude_input", "True",
            "${file}"
        ],
    }
]
```

More custom tasks can be added simply appending more of the same blocks to the [tasks.json](https://code.visualstudio.com/docs/editor/tasks#_custom-tasks) file.

## TODOs and Further development

Few ideas I'd like to experiment with in the future are:

- implement a different Dockerfile that the one from jupyter-devbox, for example creating my own Dockerfile starting from the [Jupyter Docker Stacks](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html).
- write a VSCode extension to have additional commands and shortcuts in a contextual menu (i.e., right-clicking on a notebook file in the VSCode file explorer)

## References

Here's a list of some of the references used throughout this project:

- [Developing inside a Container](https://code.visualstudio.com/docs/devcontainers/containers), help page from VSCode official docs
- Jupyter Notebooks in VScode [link]()
- jupyter-devbox [GitHub repository](https://github.com/jakoch/jupyter-devbox)
- Jupyter Docker Stacks [official documentation](https://jupyter-docker-stacks.readthedocs.io/en/latest/index.html)
- Allison Thackston [VSCode, Docker, and Github Pages](https://www.allisonthackston.com/articles/vscode-docker-github-pages.html)
- Allison Thackston [Github Pages Workspace Template](https://github.com/athackst/vscode_website_workspace)
