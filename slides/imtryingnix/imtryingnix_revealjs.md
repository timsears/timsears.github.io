---
title: I am trying nixos
subtitle: A user (in)experience report
date: October 16, 2014
author: Tim Sears
excerpt: I am trying nixox. Only 10 days in.
---

<!--
Build Notes. Need pandoc, revealjs, plus a custom template in reveal.js/. That's it.
Adapted from a blog piece, which was adapted from an older presentation.

BEGIN OLD
$ pandoc -t html5 -V revealjs-url=./reveal.js --section-divs --no-highlight -V hlss=zenburn --template=reveal.js/template-revealjs.html  -V theme=black imtryingnix_revealjs.md -o imtryingnix.html
END OLD

BEGIN NEW
Must have reveal.js (directory) at ../reveal.js
(So it can be shared with other presentations.)
Must have template file in reveal.js. It's not distributed with reveal.js (Checking it into branch timtweaks of my fork of revealjs)
After setting up blog repo to support slides AND modifying template to find the reveal.js directory....

`cd ~/blog/slides`

`mkdir imtryingnix/`

Edit imtryingnix.md>
`pandoc -t html5 -V revealjs-url=../reveal.js --section-divs --no-highlight -V hlss=zenburn --template=../reveal.js/template-revealjs.html  -V theme=black imtryingnix_revealjs.md -o imtryingnix.html`

Outputs `imtryingnix.html` in this directory.

Now it can be used as a part of the site.

-->


<style type="text/css">
p { text-align: left; }
/*
.reveal .slide h1 { font-size: 90%; text-decoration: underline; }
.reveal .slide h2 { font-size: 80%; text-decoration: underline; }
*/
</style>

## My setup

### I have been running ubuntu 12.04 on VMWare Fusion.

I started using nixos on October 6th.

_That's just 10 days ago._

###

_So all opinions herein are subject to change._

## My current setup

## has its pluses...

* Ubuntu is very stable, never crashes. I have never felt pressure to upgrade from 12.04.
* Security upgrades have never broken anything.
* `sudo apt-get install somepackage` rarely fails to provide what I need.
xs
* Binary/libc compatible with our servers in the
  cloud. Easy deployment on minimially configured production box.


## ...and its minuses

* ghc is _always_ behind in the package manager (ubuntu 12.04 ~ ghc
  7.4). I want to work on projects that require ghc 7.8.
* My dev box is used for building production code.
* Only *my* code is should break a build. Therefore I don't want to
  install software that could break my code.
* Setting up a dev box for a new person requires reading lots of old and
  potentially outdated notes to collect and install all sorts of dependencies
  together on a new box.
* The new cabal sandboxes work, but lots of code gets recompiled and
  reinstalled on every project.
* It's "hard" to run multiple versions of ghc. (I might just be lazy here.)

## Some suggestions I have heard

###

> "Bro, you're running a mac.
>  Just use brew."

I have tried everything on mac. Remember fink, ports?

brew is great. It makes make osx 95% like linux.

(Not nearly close enough for me.)

###

> "Dude, you like the bleeding edge. \
>  Try Arch."

No, it just looks that way because I use Haskell. :-)

I did run Arch for over a year. Arch is fun, has a great community and
a fantastic wiki...

...but if you want to update one thing you need to update  everything. You tend to run only the latest software and it is hard to roll back.

I broke my system more than once.

## Why Nixos? #

###

### As a Haskeller how could I resist this kind of marketing?

"Nixos: The **Purely Functional** Linux Distribution*

###

"But wait there's more!"

###

"Nixos is built on top of the Nix package manager, and is *completely  declarative*"

### Whoa, sold. { bg_color=skyblue }
![](images/sold.gif)


## What this means

###

The potential for...

* Alternative environments residing safely on the same box.
* Encoding the configuration of a development box suitable for my
company's project.
* _Calculating_ an entire machine configuration and deploy it to the cloud. (nixops)
* Specifying all dependencies for a project so that another dev can
  install it in one step.

### Let's try it out...

## Install

###

Boot a new machine from a .iso file, then...

~~~
$ fdisk /dev/sda # enter options n,p,1,w in sequence
$ mkfs.ext4 -j -L nixos /dev/sda1
$ mount LABEL=nixos /mnt
$ nixos-generate-config --root /mnt
$ nano /mnt/etc/nixos/configuration.nix # copy/paste a configuration.nix.
$ nixos-install
$ reboot
~~~

While it is truly possible to get a system going in one step, I edited the
configuration about 20 times before I had it just right for my dev box. If you
break something the old configuration is still available from the boot
menu. Tweaking is nearly risk-free.

###

Here's my config file...

~~~
[timsears@nixos:~/blog/drafts]$ cat /etc/nixos/configuration.nix
# Edit this configuration file to define what should be installed on
# your system.  Help is available in the configuration.nix(5) man page
# and in the NixOS manual (accessible by running ‘nixos-help’).

{ config, pkgs, ... }:

{
  imports =
    [ # Include the results of the hardware scan.
      ./hardware-configuration.nix
    ];

  # Use the GRUB 2 boot loader.
  boot.loader.grub.enable = true;
  boot.loader.grub.version = 2;
  # Define on which hard drive you want to install Grub.
  boot.loader.grub.device = "/dev/sda";

  swapDevices = [ { device = "/swapfile1"; } ];

  networking.hostName = "nixos"; # Define your hostname.
  # networking.wireless.enable = true;  # I'm on a vm. Not needed.

  # Select internationalisation properties.
  i18n = {
    consoleFont = "lat9w-16";
    consoleKeyMap = "us";
    defaultLocale = "en_US.UTF-8";
  };

  environment.systemPackages = with pkgs; [
    (pkgs.texLiveAggregationFun { paths = [ pkgs.texLive pkgs.texLiveExtra pkgs.texLiveBeamer ]; })
     wget
     curl
     emacs
     xlibs.xmessage
     xfce.terminal
   ];

  nixpkgs.config.allowUnfree = true;

  time.timeZone = "US/Pacific";
  security.sudo.wheelNeedsPassword = false;

  # List services that you want to enable:

  # Enable the OpenSSH daemon.
  services.openssh.enable = true;

  services.xserver = { displayManager.slim.autoLogin = true;
                       displayManager.slim.defaultUser = "timsears";
                       windowManager.awesome.enable = true;
                       windowManager.default = "awesome";
                       # only seem to affect login screen
                       virtualScreen = { x = 1920; y = 1200; };
                       resolutions = [{ x = 1920; y = 1200; }] ;
                       enable = true;
                       layout = "us";
                        };

  # Define a user account. Don't forget to set a password with ‘passwd’.
  users.extraUsers.guest = {
    name = "timsears";
    group = "users";
    extraGroups = ["wheel"];
    uid = 1000;
    createHome = true;
    home = "/home/timsears";
    shell = "/run/current-system/sw/bin/bash";
   };
}
~~~

## Configure

### There are at least three levels of configuration.

### Global

~~~~
nano /etc/nixos/configuration.nix # make changes
nixos-rebuild switch
~~~~

It's really about that simple.

### User profiles

You are encouraged to install software as a user with

~~~
`nix-env -i somepackage`
~~~

See installed packages with `nix-env -q`.

Here's mine so far...

~~~~
cabal-install-1.16.0.2
cabal2nix-1.61
chromium-stable-with-plugins-35.0.1916.114
ghc-7.6.3-wrapper
git-1.9.4
gnome-terminal-3.10.2
haskell-hakyll-ghc7.6.3-4.5.1.0
haskell-haskell-platform-ghc7.6.3-2013.2.0.0
haskell-hoogle-ghc7.6.3-4.2.32
hasktags-0.68.7
nix-repl-1.7-1734e8a
structured-haskell-mode-1.0.2
w3m-0.5.3
xkill-1.0.4
xpdf-3.03
~~~~

Eventually I will convert that to a nix expression. I expect to write
a few custon expressions to install the odd missing package (like easyVision).

You can have multiple profiles per user. (I haven't used that  feature yet.)

### Project-level

This is a major win, unique to nix.

You can customize the build environment for each project.

You can put a nix expression in a project directory.

### Project-level

Here's a file that lives in a project I have  been working on:
`~/timsears/code/probmonad/default.nix`.

This corresponds 1-to-1 with a `.cabal` file:

~~~~
{ pkgs ? (import <nixpkgs> {})
 #, haskellPackages ? pkgs.haskellPackages_ghc763
, haskellPackages ? pkgs.haskellPackages_ghc782
}:

let inherit (haskellPackages) cabal cabalInstall
    distributive comonad; # Haskell dependencies here
in

haskellPackages.cabal.mkDerivation (self: {
  pname = "probmonad";
  version = "0.1.0.0";
  src = "./.";
  isLibrary = false;
  isExecutable = true;
  buildTools = with haskellPackages; [cabalInstall];
  buildDepends = with haskellPackages; [ comonad distributive];
  meta = {
    license = self.stdenv.lib.licenses.gpl3;
    platforms = self.ghc.meta.platforms;
  };
  noHaddock = true;
})
~~~~

## Why this is great {fragment=fade-in target=p}

The default.nix file is mostly generated by a utility `cabal2nix`.

I can comment out one line to switch ghc versions distributions. I just enter `nix-shell` to drop into  the chosen environment. **Other projects are unaffected.**

The first time you switch versions nix downloads and caches all the
necessary packages. (Lazy evaluation!)

I can even hide my default environment to make sure there are no hidden dependencies in my build expression.

Dependencies can be _anything_. Nix will install them for you if there is a nix expression available.

## Impressions so far {fragment=fade-in target=li}

###

### The "Good"

* Switching environments works well and fast after the initial
  setup. It's really the first time I have been happy doing that.
* The entire set of package expressions is in one git repo. That's cool.
* You literally edit one file to configure your box globally. That's
  really cool.
* Rolling back a bad configuration is really easy.

### The "Bad"

* You *must* learn how nix expressions work.
* The docs aren't great yet. You must read the code!
* Nobody told me to read the manual's Appendix B first to see how to configure your
  system. There, I told you.
* Sometimes blogs are more helpful than the docs. Sheesh.

### The "I Don't Know Yet"

* nixops looks powerful. I want to try it.
* Some things are still arcane or rough around the edges (installing latex).
* Will I be able to write an expression for Intel IPP? It's got a
  weird installer.

# Stay Tuned
