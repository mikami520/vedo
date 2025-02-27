.. image:: https://user-images.githubusercontent.com/32848391/46815773-dc919500-cd7b-11e8-8e80-8b83f760a303.png

A python module for scientific analysis of 3D objects and
point clouds based on [VTK](https://www.vtk.org/) and [numpy](http://www.numpy.org/).

## Install and Test
```bash
pip install vedo
# Or, install the latest development version:
pip install -U git+https://github.com/marcomusy/vedo.git
```
Then
```python
import vedo
vedo.Cone().show(axes=1).close()
```
.. image:: https://vedo.embl.es/images/feats/cone.png


## Command Line Interface
The library includes a **C**ommand **L**ine **I**nterface.
Type for example in your terminal:

```bash
vedo --help
vedo https://vedo.embl.es/examples/data/panther.stl.gz
```
.. image:: https://vedo.embl.es/images/feats/vedo_cli_panther.png

Pressing `h` will then show a number of options to interact with your 3D scene:

     ==========================================================
    | Press: i     print info about selected object            |
    |        I     print the RGB color under the mouse         |
    |        <-->  use arrows to reduce/increase opacity       |
    |        w/s   toggle wireframe/surface style              |
    |        p/P   change point size of vertices               |
    |        l     toggle edges visibility                     |
    |        x     toggle mesh visibility                      |
    |        X     invoke a cutter widget tool                 |
    |        1-3   change mesh color                           |
    |        4     use data array as colors, if present        |
    |        5-6   change background color(s)                  |
    |        09+-  (on keypad) or +/- to cycle axes style      |
    |        k     cycle available lighting styles             |
    |        K     cycle available shading styles              |
    |        A     toggle anti-aliasing                        |
    |        D     toggle depth-peeling (for transparencies)   |
    |        o/O   add/remove light to scene and rotate it     |
    |        n     show surface mesh normals                   |
    |        a     toggle interaction to Actor Mode            |
    |        j     toggle interaction to Joystick Mode         |
    |        u     toggle perspective/parallel projection      |
    |        r     reset camera position                       |
    |        C     print current camera settings               |
    |        S     save a screenshot                           |
    |        E     export rendering window to numpy file       |
    |        q     return control to python script             |
    |        Esc   abort execution and exit python kernel      |
    |----------------------------------------------------------|
    | Mouse: Left-click    rotate scene / pick actors          |
    |        Middle-click  pan scene                           |
    |        Right-click   zoom scene in or out                |
    |        Cntrl-click   rotate scene                        |
     ==========================================================

### export your 3D scene to file
You can export it to a vedo file, which is actually a normal `numpy` file by pressing `E`
in your 3D scene, the you can interact with it normally using for example the key bindings shown above.

Another way is to export to a template htlm web page by pressing `F` using `x3d` backend.
You can also export it programmatically in `k3d` from a jupyter notebook.


### file format conversion
You can convert on the fly a file (or multiple files) to a different format with
```bash
vedo --convert bunny.obj --to ply
```

### some useful bash aliases
```bash
alias vr='vedo --run '        # to search and run examples by name
alias vs='vedo -i --search '  # to search for a string in examples
alias ve='vedo --eog '        # to view single and multiple images
alias vv='vedo -bg blackboard -bg2 gray3 -z 1.05 -k glossy -c blue9'
```

## Running in a Jupyter Notebook
To use in jupyter notebooks use the syntax `vedo.Plotter(backend='...')`,
you may want to install the
[k3d](https://github.com/K3D-tools/K3D-jupyter) library with:

`pip install k3d==2.7.4` (only this version is supported at present)

Other supported backend for visualization are:

- [ipyvtklink](https://github.com/Kitware/ipyvtklink) (allows interaction with the scene)
- [itkwidgets](https://github.com/InsightSoftwareConsortium/itkwidgets)
- [ipygany](https://github.com/QuantStack/ipygany)
- [panel](https://panel.holoviz.org/)
- `None`, in this case a normal graphics rendering window will pop up.

Check for more examples in the
[repository](https://github.com/marcomusy/vedo/tree/master/examples/notebooks).

## Running on a Server
- Install `libgl1-mesa` and `xvfb` on your server:
```bash
sudo apt install libgl1-mesa-glx libgl1-mesa-dev xvfb
pip install vedo
```

- Execute on startup:
```bash
set -x
export DISPLAY=:99.0
which Xvfb
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
sleep 3
set +x
exec "$@"
```

- You can save the above code above as `/etc/rc.local` and use `chmod +x` to make it executable.
    It may throw an error during startup. Then test it with, e.g.:
```python
import vedo
plt = vedo.Plotter(offscreen=True, size=(500,500))
plt.show(vedo.Cube()).screenshot('mycube.png').close()
```

## Running in a Docker container
You need to set everything up for offscreen rendering: there are two main ingredients

- `vedo` should be set to render in offscreen mode
- guest OS in the docker container needs the relevant libraries installed
    (in this example we need the Mesa openGL and GLX extensions, and Xvfb to act as a virtual screen.
    It's maybe also possible to use OSMesa offscreen driver directly, but that requires a custom
    build of VTK).

- Create a `Dockerfile`:
```bash
FROM python:3.8-slim-bullseye

RUN apt-get update -y \
  && apt-get install libgl1-mesa-dev libgl1-mesa-glx xvfb -y --no-install-recommends \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
  && rm -rf /var/lib/apt/lists/*
RUN pip install vedo && rm -rf $(pip cache dir)
RUN mkdir -p /app/data

WORKDIR /app/
COPY test.py set_xvfb.sh /app/
ENTRYPOINT ["/app/set_xvfb.sh"]
```

- `set_xvfb.sh`:
```bash
#!/bin/bash
set -x
export DISPLAY=:99.0
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
#sleep 3
set +x
exec "$@"
```

- `test.py`:
```python
from vedo import Sphere, Plotter, settings
settings.screenshotTransparentBackground = True
sph = Sphere(pos=[-5, 0, 0], c="r")
plt = Plotter(interactive=False, offscreen=True)
plt.show(sph)
plt.screenshot("./data/out.png", scale=2).close()
```

Then you can

1. `$ docker build -t vedo-test-local .`
2. `$ docker run --rm -v /some/path/output:/app/data vedo-test-local python test.py` (directory `/some/path/output` needs to exist)
3. There should be an `out.png` file in the output directory.


## Generate a single executable file
You can use [pyinstaller](https://pyinstaller.readthedocs.io/en/stable/)
to generate a single, portable, executable file for different platforms.

Write a file `myscript.spec` as:
```python
# -*- mode: python ; coding: utf-8 -*-
#
from vedo import installdir as vedo_installdir
import os
vedo_installdir = os.path.join(vedo_installdir,'fonts')

block_cipher = None

added_files = [
    (vedo_installdir+'/*', 'vedo/fonts/'),
]

a = Analysis(['myscript.py'],
             pathex=[],
             binaries=[],
             hiddenimports=[
                 'vtkmodules',
                 'vtkmodules.all',
                 'vtkmodules.util',
                 'vtkmodules.util.numpy_support',
                 'vtkmodules.qt.QVTKRenderWindowInteractor',
             ],
             datas = added_files,
             hookspath=[],
             hooksconfig={},
             runtime_hooks=[],
             excludes=[],
             win_no_prefer_redirects=False,
             win_private_assemblies=False,
             cipher=block_cipher,
             noarchive=False)
pyz = PYZ(a.pure, a.zipped_data,
             cipher=block_cipher)
exe = EXE(pyz,
          a.scripts,
          a.binaries,
          a.zipfiles,
          a.datas,
          [],
          name='stager',
          debug=False,
          bootloader_ignore_signals=False,
          strip=False,
          upx=True,
          upx_exclude=[],
          runtime_tmpdir=None,
          console=True,
          disable_windowed_traceback=False,
          target_arch=None,
          codesign_identity=None,
          entitlements_file=None )

```
then run it with
```bash
pyinstaller myscript.spec
```
See also an example [here](https://github.com/marcomusy/welsh_embryo_stager/blob/main/stager.spec).


.. include:: ../docs/tutorials.md

## Getting more help
Check out the [**Github repository**](https://github.com/marcomusy/vedo)
for more information, where you can ask questions and report issues.
You are also welcome to post specific questions on the [**image.sc**](https://forum.image.sc/) forum,
or simply browse the [**examples gallery**](https://vedo.embl.es/#gallery).


## API Documentation
Use this page to search and inspect `vedo` sub-modules, methods and functions.
These documentation pages are automatically generated
by [pdoc](https://pdoc3.github.io/pdoc/) from the source files.
