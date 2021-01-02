# Workarounds for ARM-based Apple-Silicon Mac
This is how I get most of my configurations work with MacBook Pro (13, M1, 2020)
Tested on macOS Big Sur (11.0.1)
Created on Nov 17, 2020 Last update: Jan 2, 2021

## Updates
A new version of homebrew has been released that supports Apple-Silicon Mac natively. You can install the latest version (2.7.1 or above) and get things up and running. Some packages are now offering prebuilt bottles for arm macs. You can just download and pour the bottles instead of building all the packages from source. Some packages might still require you to build them from source, but there has been few compatibility issues. The whole process is much easier now compared to a few months ago when I first wrote this guide. 

Docker has also released their [tech preview](https://docs.docker.com/docker-for-mac/apple-m1/) for ARM mac, and I can get most of my daily tasks up and running with it.

## Rosetta2
At the time of the writing (Nov 17), most of the applications are not (yet) supported by this new architecture and most of the applications will run with Rosetta2 created by Apple. 

The first time you launches a x86-64 application on your mac, it will automatically pop up and ask you to install Rosetta. The process takes about 2 minutes. And then I am able to open those applications as if they are native built for ARM mac. 

Rumors said that it takes 20 secs to load Microsoft Word at the first launch for the translation; however, it only takes 6.89 secs to open on my machine, which is as fast as, if not faster, than the loadtime on my 16 inch MacBook Pro. I don't know if it's because it's already translated when I was installing the app or Microsoft/Apple has done some optimization. For other apps, e.g. IINA, that runs with x86, even the first launch is as quick as you would expect on a new machine, most of which are under 5 secs. It's quite impressive.  

## Xcode build
If choose to use the git provided by xcode, after installing Xcode, you will have to agree with their license
``` bash
You have not agreed to the Xcode license agreements. You must agree to both license agreements below in order to use Xcode.

Hit the Return key to view the license agreements at '/Applications/Xcode.app/Contents/Resources/English.lproj/License.rtf'

Agreeing to the Xcode/iOS license requires admin privileges, please run “sudo xcodebuild -license” and then retry this command.
```
Simply use its command, press space to continue and q to exit, and then type agree to agree the license. 
Then commands like git, make will be available. 

## Homebrew
[Homebrew](https://github.com/Homebrew/brew) is not (yet) officially supported on Apple Silicon Macs, and their teams are working on it.  
They have provided a detailed list of how core formulaes are supported. You can find that list [here](https://github.com/Homebrew/brew/issues/7857).

As there's no official support for this mac at this moment, most of the installations listed below are just my personal workarounds tested on my local machine. There's no guarantee that it's the best practice or it will work on your machine.

### Installation
I found this alternative install method on the official documentation of homebrew, [which can be found here](https://docs.brew.sh/Installation#alternative-installs) 　

For ARM-based macs, their recomendation is to "do yourself a favour and install to /opt/homebrew on macOS ARM"
One of the easiest way to do that is to untar it to your local directory and then move it to /opt as it requires root permission to write in the /opt folder.
``` bash
cd ~
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
sudo mv homebrew /opt/homebrew
```

This way homebrew will be installed to the /opt/homebrew folder. We can then initiate homebrew by cd to that directory and run brew in the bin folder. 

``` bash
cd /opt/homebrew/bin
./brew update
```
This operation requires git, and git is not built-in in ARM Big Sur as it did for older macs. If you didn't install xcode beforehand, there will be another pop up that asks you to install command-line tools for developers.
Expected outputs (as on my local machine):
```
Initialized empty Git repository in /opt/homebrew/.git/
remote: Enumerating objects: 46, done.
remote: Counting objects: 100% (46/46), done.
remote: Compressing objects: 100% (37/37), done.
remote: Total 159587 (delta 18), reused 26 (delta 8), pack-reused 159541
Receiving objects: 100% (159587/159587), 40.85 MiB | 11.00 MiB/s, done.
Resolving deltas: 100% (118220/118220), done.
From https://github.com/Homebrew/brew
==> Tapping homebrew/core
Cloning into '/opt/homebrew/Library/Taps/homebrew/homebrew-core'...
remote: Enumerating objects: 753, done.
remote: Counting objects: 100% (753/753), done.
remote: Compressing objects: 100% (500/500), done.
remote: Total 835291 (delta 380), reused 446 (delta 253), pack-reused 834538
Receiving objects: 100% (835291/835291), 332.87 MiB | 11.64 MiB/s, done.
Resolving deltas: 100% (562744/562744), done.
Tapped 2 commands and 5344 formulae (5,629 files, 365.2MB).
Already up-to-date.
```
After that you can add brew (and the packages managed by brew) to your PATH.

For macOS Big Sur, the default shell is zsh. You can create a .zshrc file in your home directory and add this line to the config.
``` zsh
export PATH="/opt/homebrew/bin:$PATH"
```
### Formula
Brew has not yet provided packages precompiled for arm macs. The best way to install is with the `brew install -s <fomula>` command. As explained by homebrew, it will "Compile formula from source even if a bottle is provided. Dependencies will still be installed from bottles if they are available."
The list of supported packages can be found [here](https://github.com/Homebrew/brew/issues/7857)

I've successfully compiled several binaries on my local machine, and I will upload the binaries as bottles to save some compile time. 

List of the packages I've successfully compiled
```
autoconf	gnutls		libmpc		luarocks	sqlite
automake	guile		libssh2		mpfr		texinfo
bdw-gc		isl		libtasn1	msgpack		tree-sitter
cmake		jansson		libtermkey	ncurses		unbound
coreutils	jpeg		libtiff		nettle		unibilium
curl		libatomic_ops	libtool		openssl@1.1	wget
expat		libevent	libunistring	p11-kit		xz
gdbm		libffi		libuv		pcre2		zlib
gettext		libgit2		libvterm	pkg-config
git		libgpg-error	little-cms2	python@3.9
gmp		libiconv	lua		readline
gnu-sed		libidn2		luajit		sphinx-doc
```

Problems I've encountered:
~~While compiling neovim, I found that the system fails to recogonize my luajit installation and keeps sending me the error:
`Lua interpreter not found at /opt/homebrew/opt/luajit`~~

~~I solved this issue by creating a symlink within the `/opt/homebrew/opt/luajit/bin` folder that redirects lua to luajit,~~
```bash
cd /opt/homebrew/opt/luajit/bin
ln luajit-2.1.0-beta3 lua
```

~~However, even after this issue is fixed, I still get the error:~~
```
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/vimball && /opt/homebrew/Cellar/cmake/3.18.4/bin/cmake -E copy_directory /tmp/neovim-20201117-74316-1uv0si9/runtime/pack/dist/opt/vimball /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/vimball
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/matchit && /opt/homebrew/Cellar/cmake/3.18.4/bin/cmake -E copy_directory /tmp/neovim-20201117-74316-1uv0si9/runtime/pack/dist/opt/matchit /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/matchit
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime && /opt/homebrew/Cellar/cmake/3.18.4/bin/cmake -E copy_directory /tmp/neovim-20201117-74316-1uv0si9/runtime/doc doc
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/vimball && /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ doc -c quit
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime/pack/dist/opt/matchit && /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ doc -c quit
/bin/sh: line 1: 77744 Killed: 9               /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ doc -c quit
/bin/sh: line 1: 77743 Killed: 9               /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ doc -c quit
make[2]: *** [runtime/pack/dist/opt/matchit/doc/tags] Error 137
make[2]: *** Waiting for unfinished jobs....
make[2]: *** [runtime/pack/dist/opt/vimball/doc/tags] Error 137
cd /tmp/neovim-20201117-74316-1uv0si9/build/runtime && /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ ++t\ doc -c quit
/bin/sh: line 1: 77746 Killed: 9               /tmp/neovim-20201117-74316-1uv0si9/build/bin/nvim -u NONE -i NONE -e --headless -c helptags\ ++t\ doc -c quit
make[2]: *** [runtime/doc/tags] Error 137
make[1]: *** [runtime/CMakeFiles/runtime.dir/all] Error 2
make: *** [all] Error 2
```
 Neovim now builds with [this pull request](https://github.com/neovim/neovim/pull/12624)

Seeing that I cannot build neovim, I turned to emacs,

The prebuilt version for x86 cannot be run using Rosetta. It crashes at launchtime,

When I try to compile emacs myself, I failed yet again,

```
checking for xcrun... xcrun
checking for make... yes
checking for GNU Make... make
checking build system type... arm-apple-darwin20.1.0
checking host system type... arm-apple-darwin20.1.0
configure: error: Emacs does not support 'arm-apple-darwin20.1.0' systems.
If you think it should, please send a report to bug-gnu-emacs@gnu.org.
Check 'etc/MACHINES' for recognized configuration names.
```
But it did help me to test yet another bunch of packages. 
Ironically, the emacs.app (emacsformacosx) [downloaded from GNU website](https://emacsformacosx.com/download/emacs-builds/Emacs-27.1-1-universal.dmg) works, guess I will stick to it before neovim or emacs builds natively.

Docker desktop 3.0 has just released, but Apple silicon is still not yet supported.
## Games
Found a [great site](https://applesilicongames.com/) that has all the games that runs on Apple silicon Mac 

## Windows 
I don't miss windows at all, and I do have a windows desktoop for gaming. But I've installed a windwos VM.
Parallels Desktop is now supported on ARM mac. I tested [ACVM](https://github.com/KhaosT/ACVM) on my machine and it outperfroms SurfaceX.
