# `mac-notebook`

## Purpose

This repo contains a utility to ease the workflow of opening jupyter notebooks on a macOS box.

In a nutshell, you can open any notebook (or directory) under your home directory, in a jupyter classic notebook, from the command line; the tool will spawn a jupyter server as needed.

When requested to, the tool has a feature to run jupyter servers in virtualenvs, if it can find such virtualenvs along the way from a notebook up to your home directory. See below for details.

**NOTE:** as of now, installation is not automated.

## Usage

Once installed (see below), from a terminal you just run:

* `macnb-open` to open current directory
* `macnb-open mynotebook.ipynb` to open a notebook
* `macnb-open nb1 nb2.ipynb` to open several notebooks
* `macnb-all` displays all jupyter servers running on that box
* `macnb-list` displays the jupyter server, if running
* `macnb-kill` allows to kill the server

## Usage with virtualenvs

3 commands also exist in a `-venv` form, which triggers spotting virtualenv as
described below. So e.g. `macnb-open-venv` is equivalent to `macnb-open -e`, and same with the `list` and `kill` forms.

## Installation

You need to create symlinks in your `PATH`; for example (tweak according to your needs):

```
cd ~/git
git clone https://github.com/parmentelat/mac-notebook.git
for name in macnb-open macnb-list macnb-all macnb-kill; do
    sudo ln -sf ~/git/mac-notebook/bin/macnb /usr/local/bin/$name
    sudo ln -sf ~/git/mac-notebook/bin/macnb /usr/local/bin/$name-venv
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


## Context for the Jupyter server

The main principle behind `mac-notebook` is to spawn a Jupyter server on a
need-by-need basis. Each time a notebook is opened, a Jupyter server is
searched, and if none is found, one is spawned.

The actual context (virtualenv) used for spawning that server is thus primarily
the one that runs the first `macnb-open` after you log in, or after you
explicitly kill that server using macbn-kill.

## Virtualenvs

When enabled (i.e. when running with the -e option, or if the command name contains `venv`), `mac-notebook` uses virtualenvs as follows:

#### When opening

* starting from one notebook full path, we go up the filesystem tree up to '/'
* on the way up, we check whether we can find a `venv` subdir, that in turn looks like a jupyter-enabled virtualenv, i.e. that has a `bin/activate` and `bin/jupyter`
* if that's the case, then this is taken as the root for all notebooks underneath
* otherwise we move up to $HOME, and run all the notebooks underneath under the system jupyter.


Example:

With this imaginary filesystem layout:

```
/Users/dilbert/
  git/
    proj1/
      venv/
        bin/
          activate
          jupyter
      src/
        foo.ipynb
    proj2/
      src/
        bar.ipynb
```

* clicking on `foo.ipynb`, a jupyter server is launched in directory `/Users/dilbert/ git/ proj1/` inside a jupyter server running in the `git/proj1/venv` virtualenv;
* clicking on `bar.ipynb` results in a jupyter server being launched in directory `/Users/dilbert` using the system-wide jupyter installation.

#### Other actions

When listing or killing servers, the current directory is used, with the same algorithm, to spot or kill a given server.
