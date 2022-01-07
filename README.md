# Workarounds and Setups for ARM-based Apple-Silicon Mac

Updated with MacBook Pro (16 inch, M1 Max, 2021) Tested on macOS Monterey (12.1)

Created on Nov 17, 2020 Last update: Jan 7, 2022

## M1Max updates

Yes, I've got the new 16 inch. And this time it will replace my main workstation. So instead of a side project it will be my daily driver.

### Homebrew

Apple Silicon is now fully supported by homebrew.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Xcode commandline is required and there will be a propt for installation if it's not yet set up on your device.

However, brew is not automatically added to the path, you will have to manually add it to your zsh profile.
It's located at `~/.zprofile`. You can copy `eval "$(/opt/homebrew/bin/brew shellenv)"` and manually add it to the end of that file or use the script below.
The previous workaround by adding it to the zshrc also works

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### Brew Packages and Casks

I use brew cask to install all my applications,
You will have to agree to Xcode license before installation.

```bash
sudo xcodebuild -license accept
```

An easy way for bulk installation is to keep a list of all the applications you have and use this shell script to install every apps on the list.

It works for both packages (like python, git, neovim) and applications (steam, chrome, vscode)

An example would look like this

```bash
declare -a packages=(
  'aria2'
  'git'
  'go'
  'gpg2'
  'mas'
  'mongodb'
  'neovim'
  'node'
  'python3'
  'ruby'
  'tmux'
  'vim'
  'wine'
  'winetricks'
  'wget'
)

for pkg in "${packages[@]}"; do
  brew install "$pkg"
done

declare -a cask_apps=(
  'notion'
  'steam'
  'visual-studio-code'
 #  Other apps you have goes here
)

for app in "${cask_apps[@]}"; do
  brew install "$app"
done
```

### Shell

The terminal emulator I've been using is iTerm2, it supports arm natively and has extremely good integration with tmux and zsh.

`brew install iterm2`

The theme and fonts I use is [Nord](https://www.nordtheme.com) and [Fira Code Nerd Font](https://github.com/ryanoasis/nerd-fonts/releases/download/v2.1.0/FiraCode.zip)

I also use brew for fonts. You can install a huge selection of fonts from [cask-fonts](https://github.com/Homebrew/homebrew-cask-fonts) after adding it to your tap.

```bash
brew tap homebrew/cask-fonts
brew install font-fira-code-nerd-font
```

I've been using zsh and [prezto](https://github.com/sorin-ionescu/prezto) as the plugin manager

To install, use this command to clone prezto to your home directory

```bash
git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
```

After it's done, use this command to generate a config file.

```bash
setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
```

You can then work on `~/.zpreztorc` for plugins and settings.

A copy of my updated zpreztorc file can be found [here](https://github.com/mikelxc/ohmyconfig/blob/master/.zpreztorc). I've enabled autosuggestions, highlighting, fast directory change based on [fasd](https://github.com/clvv/fasd) and some other features.

### Keys

I use GPG keys for signing and ssh authentication.

It can be managed with GPG2 and installed using homebrew.

```
brew install gpg2
```

All the key features are fully supported on Apple Silicon.
If you do not have a key, you can use
`gpg --full-generate-key` to get a key generated.

To use it for ssh, you have to add a subkey for authentication. And use
`echo enable-ssh-support >> $HOME/.gnupg/gpg-agent.conf` to enable gpg ssh support.

To ask SSH to use gpg-agent instead of ssh-agent, you will need this in your zshrc file.

```
unset SSH_AGENT_PID
if [ "${gnupg_SSH_AUTH_SOCK_by:-0}" -ne $$ ]; then
  export SSH_AUTH_SOCK="$(gpgconf --list-dirs agent-ssh-socket)"
fi
export GPG_TTY=$(tty)
gpg-connect-agent updatestartuptty /bye >/dev/null
```

I often switch back and forth between gpg-agent and ssh-agent, so I kept it as a function.

### Keyboard Configuration

I use karabiner-elements for the keyboard configuration. This can also be installed using `brew install karabiner-elements`.

Your settings can be found in `~/.config/karabiner/assets/complex_modifications/`

### Python and ML

Python is natively supported by apple silicon macs.
However, at the time of writing, anaconda and miniconda still haven't supported arm yet.
The best available is [miniforge](https://github.com/conda-forge/miniforge#readme), a minimal installer for Conda specific to conda-forge. The installation guide can be found on their homepage, but easiest way is to install it with brew.

```bash
brew install miniforge
```

After that, you will have to initialize it for your shell.

```bash
conda init zsh
```

Afterwards, you can you conda to create environments and use it it host your packages.

### TensorFlow

This is the only major ML framework that's natively supported.

Apple even released their own support package to improve the performance. More information can be found [here](https://developer.apple.com/metal/tensorflow-plugin/)

```bash
conda install -c apple tensorflow-deps
python -m pip install tensorflow-macos
python -m pip install tensorflow-metal
```

### Java

Java 12 can be installed as a homebrew package. However, openjdk will not be automatically added to your path. To use it directly in the terminal you have to use the following command:

```bash
sudo ln -sfn /opt/homebrew/opt/openjdk/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk.jdk
echo 'export PATH="/opt/homebrew/opt/openjdk/bin:$PATH"' >> ~/.zshrc
```

### Windows Applications and Wine

Well, I don't really need windows applications to run on my mac. But I would really like to see how it performs. And it just seems to be a great challenge. And

[Wine](https://www.winehq.org) has dropped their official support for macOS. But I've been able to get their source code and get some version of it running on my mac...

For some reason, the latest verisign I've been able to compile is 6.17. However, it still need quite a lot time to get some software running. It's going to be much easier to just purchase a copy of crossover or parallels desktop.

Setups will be updated.

---

This is how I get most of my configurations work with MacBook Pro (13, M1, 2020)
Tested on macOS Big Sur (11.0.1)

## Updates

A new version of homebrew has been released that supports Apple-Silicon Mac natively. You can install the latest version (2.7.1 or above) and get things up and running. Some packages are now offering prebuilt bottles for arm macs. You can just download and pour the bottles instead of building all the packages from source. Some packages might still require you to build them from source, but there has been few compatibility issues. The whole process is much easier now compared to a few months ago when I first wrote this guide.

Docker has also released their [tech preview](https://docs.docker.com/docker-for-mac/apple-m1/) for ARM mac, and I can get most of my daily tasks up and running with it.

## Rosetta2

At the time of the writing (Nov 17), most of the applications are not (yet) supported by this new architecture and most of the applications will run with Rosetta2 created by Apple.

The first time you launches a x86-64 application on your mac, it will automatically pop up and ask you to install Rosetta. The process takes about 2 minutes. And then I am able to open those applications as if they are native built for ARM mac.

Rumors said that it takes 20 secs to load Microsoft Word at the first launch for the translation; however, it only takes 6.89 secs to open on my machine, which is as fast as, if not faster, than the loadtime on my 16 inch MacBook Pro. I don't know if it's because it's already translated when I was installing the app or Microsoft/Apple has done some optimization. For other apps, e.g. IINA, that runs with x86, even the first launch is as quick as you would expect on a new machine, most of which are under 5 secs. It's quite impressive.

## Xcode build

If choose to use the git provided by xcode, after installing Xcode, you will have to agree with their license

```bash
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

```bash
cd ~
mkdir homebrew && curl -L https://github.com/Homebrew/brew/tarball/master | tar xz --strip 1 -C homebrew
sudo mv homebrew /opt/homebrew
```

This way homebrew will be installed to the /opt/homebrew folder. We can then initiate homebrew by cd to that directory and run brew in the bin folder.

```bash
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

```zsh
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
