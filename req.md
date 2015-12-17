## Requiremens

### Base Requirements

#### Vim

vim-plug is known to work with Vim 7.0 or above.
It is strongly recommended that you use 7.3 or above.

#### Git

Most of the features should work with Git 1.7 or above.
However, installing plugins with tags is [known to fail](https://github.com/junegunn/vim-plug/issues/174) if Git is older than 1.7.10.
Git 1.8 or greater is recommended.

#### The Parallel Installers

Vim-plug supports several installers.
If more than one installer is available, they are selected in this order:

1. neovim
1. ruby
1. python
1. python3
1. viml

The parallel installers additional requirements in short are:

1. [Neovim](https://github.com/junegunn/vim-plug/wiki/faq#neovim) with job control: `exists('*jobwait')`
1. Vim with [Ruby](https://github.com/junegunn/vim-plug/wiki/faq#ruby) support: `has('ruby')` | Ruby 1.8.7 or above
1. Vim with [Python](https://github.com/junegunn/vim-plug/wiki/faq#python) support: `has('python')` or has('python3') | Python 2.6 or above

#### Neovim

The neovim job based installer is only available when using neovim.
The one caveat is that it is [not synchronous](https://github.com/junegunn/vim-plug/issues/104) like python or ruby installers.
This affects you when scripting, for instance `nvim +PlugUpdate +qa`.
In such cases vim-plug will use the python installer - which is synchronous - when available.

Neovim can be [installed](https://github.com/neovim/neovim/wiki/Installing-Neovim) on many platforms.

To use the python installer during start up, install the pypi neovim library.

```viml
sudo pip install neovim
```

If the neovim module is working, the following should work inside nvim.

```viml
:python print 'python works'
```

#### Vim (Ruby)

The ruby installer requires the ruby interpreter and vim compiled with `+ruby` support.

You can check it works inside vim by:

```viml
:ruby puts 'ruby works'
```

#### Vim (Python or Python3)

The python installer requires python version >= 2.6 or python3.
Python 2.7 is recommended as many plugins still require python2.
Vim must be compiled with `+python` or `+python3`.

To check the interfaces work:
- python: `:python print 'python 2 works'`
- python3: `python print('python 3 works')`

## Platform Installation

### Windows

On Windows the python installer works for GVim and terminal vim.
Ruby is only available when using vim from terminal.
The configurations below have been tested and work.

#### GVim

Installing GVim and required executables for Windows.

1. Insall python and put it on your PATH. For example [ActivePython 2.7](http://www.activestate.com/activepython/downloads).
2. Install git and put it on your PATH. I suggest [Git For Windows](https://git-for-windows.github.io/). If you use the portable version, remember to edit your PATH variable to make it available on the cmd prompt. If you use the installer, it can be done for you by selecting "Use Git from Windows Command Prompt".
3. Install [GVim](http://www.vim.org/download.php#pc) for Windows.
   Any version with `+python` should do.

#### msys2

Install msys2 then install the required packages.

1. Install [msys2](https://msys2.github.io/)
1. Install the required packages from terminal.
  1. Use ruby: `pacman -S vim git ruby`
  1. Use python: `pacman -S vim git python`

Reminder: The ruby installer ONLY works from terminal.

### OSX

OSX usually ships with vim and ruby so vim-plug should work out of the box.
To update to latest ruby and vim use [rvm](https://rvm.io/)
and [homebrew](http://brew.sh/) respectively.

#### Homebrew

Assuming that you have already installed RVM and Homebrew.

```sh
rvm install ruby
rvm use ruby
brew install vim
```

### Linux

#### Ubuntu, Debian & Derivatives

The default vim has support for both python and ruby installers.
Just ensure you have all the required packages.

**Ruby**:
```sh
sudo apt-get install vim git ruby
```

**Python**:
```sh
sudo apt-get install vim git python
```

#### Arch

The default vim should work, just install any missing packages.

**Ruby**:
```sh
sudo pacman -S vim git ruby
```

**Python**:
```sh
sudo pacman -S vim git python
```

#### Building Vim & Python

This step is a last resort, allowing you to build the requirements for vim-plug.
You should use this if:
  - Your version of python is too old.
  - Your version of vim doesn't have `+python` or `+python3`.
  - You need a more recent version of one that isn't available from your package manager.

It should work on any POSIX machine with the required build-tools.
Please change the dependencies line if your platform is not debian based.
By default it will install into `/usr/local`, change DIR to install elsewhere.

```bash
#!/usr/bin/env bash

# This is where python & vim will be installed to.
DIR=/usr/local
PYTHON_URL=https://www.python.org/ftp/python/2.7.10/Python-2.7.10.tgz
NEED_SUDO=
if [[ "$DIR" ==  "/usr/local"* ]]; then
  NEED_SUDO=sudo
fi

# List of dependencies on Ubuntu, bare minimum excluding GTK/GNOME support
sudo apt-get install build-essential autoconf libncurses-dev curl git

# Build python 2.7, install to $DIR
curl $PYTHON_URL | tar -xz
cd Python*
./configure --prefix=$DIR
make
$NEED_SUDO make install
cd -
command rm -rf Python*

# Build vim with +python, install to $DIR
git clone --depth 1 https://github.com/vim/vim
cd vim
vi_cv_path_python=$DIR/bin/python ./configure --prefix=$DIR --with-features=huge --enable-pythoninterp
PATH=$DIR/bin:$PATH make
$NEED_SUDO make install
cd -
command rm -rf vim
```
