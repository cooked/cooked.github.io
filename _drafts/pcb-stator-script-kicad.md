---
layout: default
#title: Test / Prova
description: Prot
---

## papers to read

see other post on axial-flux motor design

https://pcbstator.com/


## what plugins folder on the path?

see [Kicad doc](https://dev-docs.kicad.org/en/python/pcbnew/) for hints on locations

```bash

> import pcbnew; print(pcbnew.PLUGIN_DIRECTORIES_SEARCH)

/app/share/kicad/scripting
/app/share/kicad/scripting/plugins
/home/stefano/.var/app/org.kicad.KiCad/config/kicad/6.0/scripting
/home/stefano/.var/app/org.kicad.KiCad/config/kicad/6.0/scripting/plugins
/home/stefano/.var/app/org.kicad.KiCad/data/kicad/6.0/scripting
/home/stefano/.var/app/org.kicad.KiCad/data/kicad/6.0/scripting/plugins
/home/stefano/.var/app/org.kicad.KiCad/data/kicad/6.0/3rdparty/plugins
```

very different form the ones described only probably because we install from snap

## how to write kicad plugins  

https://dev-docs.kicad.org/en/python/pcbnew/  

ATTENTION: if file is in a subfolder the __init__.py file is neededed (i.e. Python package)

## before tackling this look into how to parametrize the stator  

- outer diameter (board edge?)
- inner diameter (board edge?)
- location of terminals (N,S,E,W etc.)
- nr poles
- track size (i.e. max current?)
- (optional) layers to use  

not visible/editable for now:  

- track-track distance
- track-edge distance


## Pcbnew board file format and API

[file format](https://dev-docs.kicad.org/en/file-formats/sexpr-pcb/)  

https://kicad-python-python.readthedocs.io/en/latest/


## reference - script for kicad tracks  

https://jeffmcbride.net/kicad-track-layout/#part-2-curvycad

https://jeffmcbride.net/kicad-track-layout/

https://github.com/mcbridejc

