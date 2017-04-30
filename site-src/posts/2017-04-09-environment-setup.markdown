---
title: Environment Setup
author: Oscar Vargas Torres
description: Blogging with Hakyll. Hello Spacemacs with Intero.
tags: Hakyll, Spacemacs, environment
---

# Blogging with Hakyll and Spacemacs

Hakyll is a Haskell library for generating static sites, like this blog. Thanks to Pandoc it supports markdown, $\TeX$ and Emacs *org mode*. 

The structure of the source code for this blog is as follows:
```text
$ tree -L 3 -C -I '_cache|_site|TAGS|ltximg'
.
├── app
│   └── Site.hs
├── blog.cabal
├── LICENSE
├── README.md
├── Setup.hs
├── site-src
│   ├── index.html
│   ├── posts
│   │   ├── 2017-04-09-environment-setup.markdown
│   ├── static
│   │   ├── 404.markdown
│   │   ├── about.markdown
│   │   ├── css
│   │   ├── favicon.ico
│   │   ├── MathJax
│   │   └── talks.markdown
│   └── templates
│       ├── cloud.html
│       ├── default.html
│       ├── post.html
│       ├── post-item.html
│       ├── posts.html
│       └── tag.html
└── stack.yaml
```

## Source code for the blog.

Source code is backed up in a [personal github repository](https://github.com/oscarvarto/haskellBlog)

# Developing with Linux

At the moment I am using [Antergos](https://antergos.com/) Linux, a Linux distribution based on [Arch Linux](https://www.archlinux.org/).

I chose the following repository entries for my `/etc/pacman.conf`. For more information on this, please read the [ArchHaskell wiki entry].

```text
[antergos]
SigLevel = PackageRequired
Include = /etc/pacman.d/antergos-mirrorlist

#[testing]
#Include = /etc/pacman.d/mirrorlist

[core]
Include = /etc/pacman.d/mirrorlist

[extra]
Include = /etc/pacman.d/mirrorlist

[haskell-core]
Server = http://xsounds.org/~haskell/core/$arch

[haskell-happstack]
Server = http://noaxiom.org/$repo/$arch

#[community-testing]
#Include = /etc/pacman.d/mirrorlist

[community]
Include = /etc/pacman.d/mirrorlist

# If you want to run 32 bit applications on your x86_64 system,
# enable the multilib repositories as required here.

#[multilib-testing]
#Include = /etc/pacman.d/mirrorlist

[multilib]
Include = /etc/pacman.d/mirrorlist
```

It was necessary to add some keys. See the [ArchHaskelli wiki entry] entry.

After `pacman -Syu` I installed ghc version 8.0.2, and some other packages from haskell-core repository.

[ArchHaskell wiki entry]: https://wiki.archlinux.org/index.php/ArchHaskell

## `blog.cabal`

The most important part was the `build-depends` configuration.

```cabal
name:                blog
version:             0.1.0.0
synopsis:            My personal blog, generated using Hakyll 
homepage:            https://github.com/oscarvarto/haskellBlog
license:             Apache Licence Version 2.0
license-file:        LICENSE
author:              Oscar Vargas Torres
maintainer:          vargas.torres.oscar@gmail.com
category:            Web
build-type:          Simple
cabal-version:       >=1.10

executable blog
  hs-source-dirs:      app
  main-is:             Site.hs
  ghc-options:         -threaded -rtsopts -with-rtsopts=-N
  build-depends:       base,
                       containers,
                       hakyll,
                       temporary,
                       pandoc,
                       skylighting >= 0.1
  default-language:    Haskell2010

source-repository head
  type:     git
  location: https://github.com/oscarvarto/haskellBlog
```

## `stack.yaml` and `docker`

For the stack.yaml, the `docker` section configuration is important to keep the haskell stack + intero + spacemacs development environment working at the same time that the hakyll blog modification.

```yaml
resolver: lts-8.5

packages:
- '.'
extra-deps: []

flags: {}

extra-package-dbs: []

docker:
    enable: true
    run-args: ["--net=host"] 
```

It was necessary to [setup docker for Antergos/ArchLinux](https://wiki.archlinux.org/index.php/Docker#Installation). After installation, to add myself (`oscarvarto`) to docker group: 

```text
$ sudo gpasswd -a ${USER} docker
$ systemctl restart docker.service
```

It was also necessary to pull the `fpco/stack-build` docker image with `$ stack docker pull`.

## Build process

After that, it is possible to run:

```text
$ stack build
$ stack exec blog build
$ stack exec blog watch
``` 

and open a web browser to see my personal notes locally (http://127.0.0.1:8000) 

Once in a while, it is necessary to run

```
$ stack clean
$ stack exec blog clean
```

# Spacemacs 

I have chosen [Spacemacs](http://spacemacs.org/) for Haskell development and general text edition needs (for example, to write my markdown/org notes).

## Haskell development environment.

Spacemacs has a [Haskell layer](https://github.com/syl20bnr/spacemacs/tree/master/layers/%2Blang/haskell) with different autocompletion backends. I have chosen the `intero` backend because of its good `stack` support.

```text
[oscarvarto@lap ~]$ stack install apply-refact hlint stylish-haskell hasktags hoogle intero
```

Spacemacs' configuration layers currently has:

```commonlisp
   dotspacemacs-configuration-layers
   '(
     helm
     auto-completion
     better-defaults
     git
     syntax-checking
     version-control
     haskell
     '(auto-completion
       (haskell :variables haskell-completion-backend 'intero))
     yaml
     )
```

For `user-config`:

```commonlisp
(defun dotspacemacs/user-config ()
  "Configuration function for user code.
This function is called at the very end of Spacemacs initialization after
layers configuration.
This is the place where most of your configurations should be done. Unless it is
explicitly specified that a variable should be set before a package is loaded,
you should place your code here."

  (add-hook 'haskell-mode-hook 'intero-mode)
  (xterm-mouse-mode 1)

  ;; Defeat smartparens-mode in evil mode
  (add-hook 'evil-insert-state-entry-hook 'turn-off-smartparens-mode)
  (add-hook 'evil-insert-state-exit-hook 'turn-on-smartparens-mode)

  ;; Alternative way to defeat smartparens-mode in hybrid mode
  (add-hook 'evil-hybrid-state-entry-hook 'turn-off-smartparens-mode)
  (add-hook 'evil-hybrid-state-exit-hook 'turn-on-smartparens-mode)
  )
```

For syntax highlighting [pandoc](https://github.com/jgm/pandoc) (which is used by Hakyll) has recently switched from highlighting-kate to [skylighting](https://github.com/jgm/skylighting)

```text
$ stack install skylighting --flag "skylighting:executable"
```
