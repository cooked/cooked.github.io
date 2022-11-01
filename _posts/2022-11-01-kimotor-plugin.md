---
layout: single
title: KiMotor - Parametric PCB motors in KiCad
tags: [KiCad, Python, PCB]
category: software
header:
    teaser: /assets/img/kimotor.png
# papers to read: see other post on axial-flux motor design
# https://pcbstator.com/
---

Notes on a proof-of-concept KiCad plugin to generate parametric PCB stators for axial-flux electric motors.

## Current status

The plugin currently allows the customization of few main mechanical and electrical parameters. The plugin is a POC of possibly good things to come. It is still a far cry from being reliable for the day to day use (see [issues](https://github.com/cooked/kimotor/issues)) but you can have a look at the code and/or download it from [GitHub](https://github.com/cooked/kimotor).

![kimotor](/assets/img/kimotor.png){: .align-center}

## Developer notes for KiCad plugins

### Pcbnew board file format and API

[file format](https://dev-docs.kicad.org/en/file-formats/sexpr-pcb/)  

[KiCad User Documentation](https://kicad-python-python.readthedocs.io/en/latest/)

### Locate plugin folders

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

I've found a very different structure from the ones described in the official documentation. This is probably due to the fact I've install KiCad from snap.

### Important details on plugins  

**ATTENTION** if the plugin file is in a subfolder the **__init__.py** file is needed (i.e. it is a proper Python package)
{: .notice--warning}

## References

[KiCad Plugins](https://dev-docs.kicad.org/en/python/pcbnew/) official documentation with hints on standard folders for plugin

[](https://github.com/mcbridejc)

[](https://jeffmcbride.net/kicad-track-layout/#part-2-curvycad)

[](https://jeffmcbride.net/kicad-track-layout/)
