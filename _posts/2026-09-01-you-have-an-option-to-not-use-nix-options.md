---
layout: post
title: "You have an option to not use Nix options"
date: 2026-09-01
categories: software thoughts project
---
Contrary to the idea of keeping all your configuration in nix, I find myself using nix options less and less. Was it a wrong choice of a distro? 

## Nix is just perfect

My Linux journey started with Arch. I daily drove this distro for a few years, so for a long time official repos and AUR were my go-to places to get new software. On several occasions, during updates or just installing new packages, I broke other parts of the system. This seems like a classic newbie mistake, but it happened often enough to be frustrating, and updating the system became a scary task that I often put off.

So naturally, when I heard about [Nix](https://nixos.org/), I was hooked immediately. It promised to solve all those problems that I faced, and it actually did. I still use Nix packages as they are an amazing source of software; it is always reliable and it has never broken anything for me. Over time, I have even learned a bit about writing my own derivations and how to do that. So, I have some experience with packaging software myself.

It is safe to say that I have had an amazing experience with Nix. I know how to use it efficiently, how to expand it for my own use cases, and I have my whole system rewritten in Nix (yes, with secrets and impermanence). So what seems to be the problem?

## Nix options

[Nix options](https://search.nixos.org/options) are the things you use to configure your system. They abstract some difficult logic; for example, you can set up your entire Linux networking stack with a few lines of Nix code. When I first started using Nix, it was mind-blowing to me that I didn't need to know all the details about system configuration to have a well-working system.

I loved this initially, and when I discovered that you can do the same with your tools by using Home Manager, I immediately migrated all my configs to the Nix spec.

Instead of being readable, well-documented, pure code with no side effects, my system config became a convoluted mess of interdependent options that influenced the behavior of other parts of the system.

But the mess wasn't the worst part. The worst part was that every tool uses its own configuration in a different format—TOML, YAML, JSON, or even custom ones—while Nix uses Nix. And this is when it gets tricky: every example on the internet is in the original configuration format. So, whenever I added or changed something, I had to find the option in the tool's docs, then find it on the Home Manager options page (no, AI is literally clueless when it comes to this), pray that it exists and does what I think, and every small tweak required a whole system rebuild. This was too impractical and time-consuming to put up with.

### Glorious packages

After watching one of [Vimjoyer's videos](https://www.youtube.com/watch?v=Zzvn9uYjQJY), I remembered why I came to Nix in the first place. It was not to add complexity to my whole setup, but to eliminate it. I remembered that packages were the thing that removed the complexity. They encapsulate the complicated packaging logic within them, which is similar to what I have created with options.

It is possible to take any program that I want to use—let's say [Hyprland](https://hypr.land/)—and create a new package out of it, but with the desired configuration baked into the package.

```nix
{pkgs, ...}:

pkgs.symlinkJoin {
  name = "Hyprland";
  paths = [pkgs.hyprland];
  buildInputs = [pkgs.makeWrapper];
  inherit (pkgs.hyprland) passthru version;
  postBuild = let
    extraPkgs = [
      pkgs.wrapped.hypridle
      # ...
      pkgs.wl-mirror
    ];
  in ''
    wrapProgram $out/bin/Hyprland \
        --suffix PATH : ${pkgs.lib.strings.makeBinPath extraPkgs} \
        --add-flags "--config $out/bin/config.lua"
    cp ${./config.lua} $out/bin/config.lua
  '';
}
```

This is what I did with most of my tools. This has several benefits. Firstly, the configuration is in the native configuration format, so you can just copy-paste from the documentation, from someone else's config, or even ask an AI, and it will most likely work. You can always take this configuration to any non-Nix system, and it will just work without many hiccups, while also preserving the convenience of a Nix package. You can even have a few different configurations of the same tool (if you want to, for some reason).

It also has the benefit of uncluttering your `.config` directory, which is usually full of random files, and makes it possible to force non-XDG-compliant applications to use standard directories, such as `.local`.

```nix
  postBuild = ''
    # Force Claude to not create config in the home directory
    wrapProgram $out/bin/claude \
        --suffix HOME "" "/.local/share/claude"
  '';
```

## I don't need a manager

This approach has changed my config so much that I decided I have no use for Home Manager anymore. I can just configure my tools via wrapped packages and provide them to my user in the environment, erasing a whole level of complexity from my configuration.

It has also changed the way I think about different configurations. At first, it was a bunch of conditional statements that had to work together to create a different final state. Right now, I retain a clear separation for both of them; whenever I have a need for a different configuration, I just duplicate the original one and change what I need. While this may not be fully DRY, it definitely is easier to understand and keeps my eyes dry when I need to add another variant.

Everything I wrote may seem very obvious. After all, Nix is a functional programming language for a reason. It was meant to transform one input into one output without creating any side effects. The side effects were introduced with Nix options, where one part of the configuration can influence another; it is not pure.

When you are riding a hype train, you often forget the "why"—the original problem you were trying to solve—and just try out the newest shiny stuff. But it's good to make those mistakes sometimes. I've picked up a lot of valuable debugging experience and general Linux knowledge along the way. I'm also not saying that you can't use Nix options; in fact, I still use them a lot. It is just important to understand when it is a good choice, and when a simple repackage is enough.
