# `mac-notebook`

## purpose

this repo contains a utility to ease the workflow of opening jupyter notebooks on a macos box

in a nutshell, you can open any notebook (or directory) under your home directory, in jupyter classic notebook, from the command line; the tool will spawn a (unique per user) jupyter server if needed

as of now, installation is not automated

## usage

once installed, from a terminal you just run:

* `macnb-open` to open current directory
* `macnb-open mynotebook.ipynb` to open a notebook
* `macnb-open nb1 nb2.ipynb` to open several notebooks
* `macnb-list` displays the (unique) jupyter server, if running
* `macnb-kill` kills that server, if running

## installation

you need to create 3 symlinks in your `PATH`; for example (tweak according to your needs)

```
cd ~/git
git clone https://github.com/parmentelat/mac-notebook.git
for name in macnb-open macnb-list macnb-kill; do
   sudo ln -s /usr/local/bin/$name ~/git/mac-notebook/bin/macnb
done
```

## finder

using Automator, you can easily create a native macos application that wraps `macnb-open`, and so have double-clicking on a `ipynb` file trigger this function.

Under automator:

* create an `Application`
* add an action `Run shell script`
* change `Pass input:` to `as arguments`
* then replace the script with (assuming you installed in `/usr/local/bin`)

```
/usr/local/bin/macnb-open "$@"
```

* save the application in `/Applications/mac-notebook`

from that point, you can select this application from the finder:

* right-click on a `.ipynb` file, select `Get Info`
* in the `Open with` area, select this newly created `mac-notebook` application,
* optionnnally select `'Change All'` if you want this mapping to apply on all `ipynb` files.