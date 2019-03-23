# `mac-notebook`

## Purpose

This repo contains a utility to ease the workflow of opening jupyter notebooks on a macOS box.

In a nutshell, the purpose is to

* be able to open any notebook (or directory) under your home directory, in a jupyter classic notebook, from the command line; the tool will spawn a jupyter server as needed;
* with a little more care, be able to double click on a notebook to achieve the same result.

## virtualenvs

The tool has a builtin feature for dealing with virtualenvs. The challenge here
is that when you double click on a notebook in your finder, there is no way you
can describe in what virtualenv you want this to happen.

So our approach relies on the following convention:

* create virtualenvs in directories named `venv`
* these should be placed right under the directory where the virtualenv applies

### Example

![](doc/example.png)

With the filesystem layout depicted above:

* clicking on `rise/notebooks/slide.ipynb` will end up running in the `rise/venv` virtualenv
* same for `specific/notebooks/notebook.ipynb`, running inside `specific/venv`
* however `usual/notebooks/regular.ipynb` will execute in the globally installed python/jupyter setup, assuming that no `venv` directory can be found on the way up to the root directory.

### Expectations, and convention about `nbextensions`

There are a few additional assumptions made by the system:

* when a `venv` directory is found, it must also - of course - have `jupyter` installed locally; a warning is issued otherwise and the virtualenv is skipped
* when no suitable `venv` can be found, then the notebook is opened inside the globally-installed jupyter/python context
* in addition, there is a more subtle point about `nbextensions`</br>
  the point here is about whether or not extensions should be shared among the various jupyter installations
  the adopted convention is that, if `venv` contains a subdirectory named `jupyter/nbextensions` then this one will be defined as `JUPYTER_DATA_DIR`, meaning that it is where jupyter nbextensions are searched for.


**NOTE:** as of now, installation is **not** automated, follow instruction
*below.

## Usage

Once installed (see below), from a terminal you just run:

* `macnb-open` to open current directory
* `macnb-open mynotebook.ipynb` to open a notebook
* `macnb-open nb1 nb2.ipynb` to open several notebooks
* `macnb-all` displays all jupyter servers running on that box
* `macnb-list` displays the jupyter server for local virtualenv, if running
* `macnb-kill` allows to kill that server

## Installation

You need to create symlinks in your `PATH`; for example (tweak according to your needs):

```
cd ~/git
git clone https://github.com/parmentelat/mac-notebook.git
ln -sf ~/git/mac-notebook/bin/macnb-bashrc /usr/local/bin
for name in macnb-open macnb-list macnb-all macnb-kill; do
    sudo ln -sf ~/git/mac-notebook/bin/macnb /usr/local/bin/$name
done
```

## Finder: create a macOS app

Using Automator, you can easily create a native macOS application that wraps
`macnb-open`, and so have double-clicking on a `ipynb` file trigger this
function.

Under automator:

* create an `Application`
* add an action `Run shell script`
* change `Pass input:` to `as arguments`
* then replace the script with (assuming you installed in `/usr/local/bin`)

```
/usr/local/bin/macnb-open "$@" &
```

* save the application in `/Applications/mac-notebook`

## Finder - bind mouse clicks on .ipynb files

From that point, you can select this application from the finder:

* right-click on a `.ipynb` file, select `Get Info`
* in the `Open with` area, select this newly created `mac-notebook` application,
* optionally select `'Change All'` if you want this mapping to apply on all `ipynb` files.

## Finder - attach jupyter icon to that app

It's nicer if your app shows the jupyter logo:

```
cp ~/git/mac-notebook/mac-notebook.icns /Applications/mac-notebook.app/Contents/Resources/AutomatorApplet.icns
```

**Devel note:** For the record, the way to generate the `.icns` file was:

* from icon/jupyter-logo-transparent.png, extract a square portion
* create the 10 variants of sizes in `icon.iconset` with these exact names
* run `iconutil --convert icns -o mac-notebook.icns icon.iconset/`
