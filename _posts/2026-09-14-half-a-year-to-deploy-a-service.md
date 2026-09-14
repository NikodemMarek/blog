---
layout: post
title: "Half a year to deploy a service"
date: 2026-09-14
categories: self-hosting software thoughts project
---
When I start in my journey of self-hosting, I didn't really know what I wanted. And I've played around with a few different solutions. This is a comparison of my experiences with all of them.

#### Proxmox mockery

Before I even started thinking about hosting something for real, I decided to give [proxmox](https://www.proxmox.com/en/products/proxmox-virtual-environment/overview) a try, as I've had a friend using it for his homelab. The documentation was not very extensive, the setup was very manual and imperative, the native linux networking setup was just a nightmare to configure and I couldn't figure out how to apply proper security measures.

It felt like a solution for people who just want to have something done, with no attention given to the details, I like to get my hands dirty, and really understand everything, from the ground up. Proxmox was a big disappointment, and I quickly repurposed it into a nix server.

#### The nix hammer

The actual self-hosting project started with the "everything via nix" approach. The setup was: service configured with nix options, deployed on bare metal, with direct access to any files or directories that it needed.

```nix
services.navidrome = {
  enable = true;
  openFirewall = true;
  settings.MusicFolder = "/mnt/music";
};
```

There wasn't too much complexity, and it just worked pretty well. Only issues were that occasionally I had to ssh jnto the machine to restart some service, and as I have discussed in [the previous blog post](https://nikodemmarek.github.io/blog/software/thoughts/project/2026/09/01/you-have-an-option-to-not-use-nix-options.html), nix options were problematic sometimes, but at least everything was declarative.

The problems started when I decided that it's time to look a bit more into security. The obvious first step was to containerize all the services, and introduce some network separation. The obvious choice back then, was to use native [nixos containers](https://wiki.nixos.org/wiki/NixOS_Containers). The containers worked, but configuration and maintenance was a nightmare. The hardest thing was networking, nix options made it so extremely difficult to set it up, as it generated a weird mix of configuration files and options, from different parts of the linux ecosystem. I could not figure out how to set up NAT, masquerading, bridges, and everything felt like it was held together with a duct tape.

#### Just one nix nail to hammer in

Then I decided to give [Kubernetes](https://kubernetes.io/) a try. At first, I almost gave up, because I wanted to set it up on nixos, with impermanence, connected via tailscale, with the proper security already in place. But I didn't really understand what Kubernetes even is, let alone how does it work. I was just struggling with the auto-generated firewall rules, impermanence deleting important configs, and the lack of proper documentation for the nix module. Before I gave up completely, I stepped back a bit, and tried Kubernetes in [minikube](https://minikube.sigs.k8s.io/docs/), to get a grip of the concepts.

So I came back to the project, but this time armed with new knowledge. I gave up on the full k8s setup, and I went with [k3s](https://k3s.io/), which is less intensive and easier to set up. It also has a pretty nice nix module. After a bit of fiddling with [tailscale](https://docs.k3s.io/networking/distributed-multicloud#integration-with-the-tailscale-vpn-provider-experimental), I managed to spin it up properly.

#### Feels like LEGO bricks

To not create too much complications for myself from the start, I went with a simple approach of having the pods connected to storage directly, like in the familiar docker compose.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: navidrome
spec:
  volumes:
    - name music
      hostPath:
        path: /mnt/music
        type: DirectoryOrCreate
  containers:
    - name: navidrome
      image: deluan/navidrome:latest
      volumeMounts:
        - name: music
          mountPath: /music
```

Then when I expanded to two notes, I realized that I needed a better solution for storage. [Longhorn](https://longhorn.io/) volumes came to rescue. It turned out to be very easy to use, I could just swap my storage setup for another one. So I emigrated everything over to longhorn, and I've been running it like this ever since.

And honestly, everything in kubernetes feels like this. It is so decoupled, that you can just swap everything out, with almost no changes to the rest of the setup. [FluxCD](https://fluxcd.io/) was also a major contributor towards this, forcing me to [write everything down](https://github.com/NikodemMarek/infra), and providing the prefect documentation for me and llms. It finally felt clean, after all the previous approaches. And for debugging, you need to know just a few commands.

Also, if you want to deploy your own apps the nix could be easily integrated with Kubernetes. It is possible to [build containers with nix](https://nixos.org/manual/nixos/stable/#sec-image-nixos-rebuild-build-image) and deploy them on Kubernetes. But since I'm deploying mostly pre-made apps, I don't need this functionality.

#### Retrospective

 Kubernetes is way better at deploying applications than nix. Nix was not made for this specifically, and it shows. The networking in Kubernetes is extremely easy. You can pick and choose components almost without any problems, and they will just work together. When you need to manage more than one service. It's definitely worth considering over bare metal, or even containerized setup. Big factor in play, when choosing the solution is community, popularity and compatibility. While AI struggles to configure anything in nix, it has absolutely no problem configuring kubernetes. Apparently kubernetes was supposed to be very difficult, but it just feels stupidly simple, when I compared it to how the other solutions felt.
