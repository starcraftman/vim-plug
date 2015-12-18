## Tips

### Automatic installation

Place the following code in your .vimrc before `plug#begin()` call

```vim
if empty(glob('~/.vim/autoload/plug.vim'))
  silent !curl -fLo ~/.vim/autoload/plug.vim --create-dirs
    \ https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
  autocmd VimEnter * PlugInstall | source $MYVIMRC
endif
```

### Loading plugins manually

With `on` and `for` options, vim-plug allows you to defer loading of plugins. But if you want a plugin to be loaded on an event that is not supported by vim-plug, you can set `on` or `for` option to an empty list, and use `plug#load(names...)` function later to load the plugin manually. The following example will load [ultisnips](https://github.com/SirVer/ultisnips) and [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) first time you enter insert mode.

```vim
" Load on nothing
Plug 'SirVer/ultisnips', { 'on': [] }
Plug 'Valloric/YouCompleteMe', { 'on': [] }

augroup load_us_ycm
  autocmd!
  autocmd InsertEnter * call plug#load('ultisnips', 'YouCompleteMe')
                     \| call youcompleteme#Enable() | autocmd! load_us_ycm
augroup END
```

### Gist as plugin

Unlike [NeoBundle](https://github.com/Shougo/neobundle.vim), vim-plug does not natively support installing small Vim plugins from Gist. But there is a workaround if you really want it.

```vim
Plug 'https://gist.github.com/952560a43601cd9898f1.git',
    \ { 'dir': g:plug_home.'/xxx/plugin', 'rtp': '..' }
```

### Portable Vimrc For Windows & UNIX

With neovim's adoption of XDG and the existing `vimfiles` vs `.vim` folders
there are now 4 possible locations.
This snippet allows the same vimrc to be used across all.
It sets `g:vim_dir` to the correct root.

```viml
let s:win_shell = (has('win32') || has('win64')) && &shellcmdflag =~ '/'
if has('nvim')
  let g:vim_dir = s:win_shell ? '$USERPROFILE/AppData/Local/nvim' : '~/.config/nvim'
else
  let g:vim_dir = s:win_shell ? '~/vimfiles' : '~/.vim'
endif

call plug#begin(expand(g:vim_dir . '/plugged'))

Plug 'junegunn/vader.vim'
" More plugins ...

call plug#end()
```

### Migrating from other plugin managers

Download plug.vim in `autoload` directory

```sh
curl -fLo ~/.vim/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

and update your .vimrc as needed.

<table>
<thead>
<tr>
<th>
With Vundle.vim
</th>
<th>
Equivalent vim-plug configuration
</th>
</tr>
<tr>
<td>
<pre>
filetype off
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'junegunn/seoul256.vim'
Plugin 'junegunn/goyo.vim'
Plugin 'junegunn/limelight.vim'
call vundle#end()
filetype plugin indent on
syntax enable
</pre>
</td>
<td>
<pre>
call plug#begin('~/.vim/plugged')
Plug 'junegunn/seoul256.vim'
Plug 'junegunn/goyo.vim'
Plug 'junegunn/limelight.vim'
call plug#end()
</pre>
</td>
</tr>
</table>

vim-plug does not require any extra statement other than `plug#begin()` and `plug#end()`.
You can remove `filetype off`, `filetype plugin indent on` and `syntax on` from your
`.vimrc` as they are automatically handled by `plug#begin()` and `plug#end()`.

Since all the other major plugin managers store plugins in "bundle" directory,
you might want to pass it to `plug#begin()` if you do not wish to reinstall plugins.

```vim
" For Mac/Linux users
call plug#begin('~/.vim/bundle')

" For Windows users
call plug#begin('~/vimfiles/bundle')
```

## FAQ

### Does vim-plug generate help tags for my plugins?

Yes, it automatically generates help tags for all of your plugins whenever you run `PlugInstall` or `PlugUpdate`. But you can regenerate help tags only with `plug#helptags()` function.

### Shouldn't vim-plug update itself on `PlugUpdate` like Vundle?

There is a separate `PlugUpgrade` command and there are valid reasons behind the decision.
A detailed discussion can be found [here](https://github.com/junegunn/vim-plug/pull/240).

If you really want to use `PlugUpdate` instead of `PlugUpgrade` you can set up vim-plug like follows:

```sh
# Manually clone vim-plug and symlink plug.vim to ~/.vim/autoload
mkdir -p ~/.vim/{autoload,plugged}
git clone https://github.com/junegunn/vim-plug.git ~/.vim/plugged/
ln -s ~/.vim/plugged/vim-plug/plug.vim ~/.vim/autoload
```

and in your .vimrc:

```vim
call plug#begin('~/.vim/plugged')
Plug 'junegunn/vim-plug'
" ...
call plug#end()
" The caveat is that you should *never* use PlugUpgrade
delc PlugUpgrade
```

Unlike `PlugUpgrade`, you'll have to restart Vim after vim-plug is updated.

### What's the deal with `git::@` in the URL?

When vim-plug clones a repository, it injects `git::@` into the URL (e.g. `https://git::@github.com/junegunn/seoul256.vim.git`) which can be an issue when you want to push some changes back to the remote. So why?

It's a little hack to avoid username/password prompt from git when the repository doesn't exist. Such thing can happen when there's a typo in the argument, or when the repository is removed from GitHub. It looks kind of silly, but doing so is the only way I know that works on various versions of git. However, Git 2.3.0 introduced `$GIT_TERMINAL_PROMPT` which can be used to suppress user prompt, and vim-plug takes advantage of it and removes `git::@` when Git 2.3.0 or above is found.

Also, there are two ways to override the default URL pattern:

1. Using full git url: `Plug 'https://github.com/junegunn/seoul256.vim.git'`
2. Or define `g:plug_url_format` for the plugins that you need to work on.

```vim
let g:plug_url_format = 'git@github.com:%s.git'
Plug 'junegunn/vim-easy-align'
Plug 'junegunn/seoul256.vim'

unlet g:plug_url_format
Plug 'tpope/vim-surround'
Plug 'tpope/vim-repeat'
```

See [#168](https://github.com/junegunn/vim-plug/issues/168), [#161](https://github.com/junegunn/vim-plug/issues/161), [#133](https://github.com/junegunn/vim-plug/issues/133), [#109](https://github.com/junegunn/vim-plug/issues/109), [#56](https://github.com/junegunn/vim-plug/issues/56) for more details.

## Troubleshooting

### Plugins are not installed/updated in parallel

Parallel installer is only enabled when at least one of the following conditions is met:

1. Vim with Ruby support: `has('ruby')` / Ruby 1.8.7 or above
1. Vim with Python support: `has('python')` / Python 2.6 or above
1. Neovim with job control: `exists('*jobwait')`

For more help, see the [requirements](https://github.com/junegunn/vim-plug/wiki/requirements).

### Vim: Caught deadly signal SEGV

If your Vim crashes with the above message, first check if its Ruby interface is
working correctly with the following command:

```vim
:ruby puts RUBY_VERSION
```

If Vim crashes even with this command, it is likely that Ruby interface is
broken, and you have to rebuild Vim with a working version of Ruby.
(`brew remove vim && brew install vim` or `./configure && make ...`)

If you're on OS X, one possibility is that you had installed Vim with
[Homebrew](http://brew.sh/) while using a Ruby installed with
[RVM](http://rvm.io/) or [rbenv](https://github.com/sstephenson/rbenv) and later
removed that version of Ruby. Thus, it is safer to build Vim with system ruby.

```sh
rvm use system
brew reinstall vim
```

If you're on Windows using cygwin and the above ruby command fails with:
`cannot load such file -- rubygems.rb (LoadError)`.
It means cygwin is missing the `rubygems` package.
Install it using the setup.exe that came with cygwin.

[Please let me know](https://github.com/junegunn/vim-plug/issues) if you can't
resolve the problem. In the meantime, you can put `let g:plug_threads = 1` in your vimrc, to disable the parallel installers.

### YouCompleteMe timeout

[YouCompleteMe (YCM)](https://github.com/Valloric/YouCompleteMe) is a huge project and you might run into timeouts when trying to install/update it with vim-plug.

The parallel installer of vim-plug (ruby or python) times out only when the stdout of the process is not updated for the designated seconds (default 60). Which means even if the whole process takes much longer than 60 seconds to complete, if the process is constantly printing the progress to stdout (`10%`, `11%`, ...) it should never time out. Nevertheless, we still experience problems when installing YCM :(

Workarounds are as follows:
- Increase `g:plug_timeout`
- Install YCM exclusively with `:PlugInstall YouCompleteMe`
    - In this case single-threaded vimscript installer, which never times out, is used
- Asynchronous Neovim installer does not implement timeout.

### fatal: dumb http transport does not support --depth

Apparently the git option `--depth 1` requires SSL on the remote Git server. It is now default, to reduce download size. To get around this, you can:

**Disable Shallow Cloning**

Add `let g:plug_shallow = 0` to your .vimrc.

**Mirror the repository on a Git server with https (i.e. Github or BitBucket).**

Then just add it normally with the new URI.

**Mark the plugin as local/unmanaged**

a) Clone it locally to `~/.vim/plugged/plugin_name`

b) Add to the vimrc with `Plug '~/.vim/plugged/plugin_name'`.

The leading tilda tells vim-plug not to do anything other than rtp for plugin_name.

### Windows System Error E484
There are two possible causes we've encountered.

1. Bad escaping. On Windows, if you use '<', '>' or '|' in the file path, vim-plug is known to fail. Any other chars should be fine.

1. System changes due to AutoRun commands executed on cmd.exe startup. See [docs](https://technet.microsoft.com/en-us/library/cc779439%28v=ws.10%29.aspx).

To see if your system suffers this second problem, run these reg queries. If either one returns a path to a bat file, you should carefully read it. You may have to edit/disable it to get vim-plug working.
```batch
REG QUERY "HKCU\Software\Microsoft\Command Processor" /v AutoRun
REG QUERY "HKLM\Software\Microsoft\Command Processor" /v AutoRun
```

### Errors on fish shell

If vim-plug doesn't work correctly on fish shell, you might need to add `set
shell=/bin/sh` to your `.vimrc`.

Refer to the following links for the details:
- http://badsimplicity.com/vim-fish-e484-cant-open-file-tmpvrdnvqe0-error/
- https://github.com/junegunn/vim-plug/issues/12
