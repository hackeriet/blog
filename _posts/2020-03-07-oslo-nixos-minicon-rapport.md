---
layout: post
title: "Oslo NixOS MiniCon 2020 rapport"
author: fnords && sgo
category: [talks, nixos]
---
On February 22. and 23. Oslo NixOS User Group hosted a mini conference at
Hackeriet. We had a variety of talks about different parts of the Nix ecosystem.

## DAY 1

### The Nix ecosystem

![Elis Hirwig](/images/nixos_minicon_2020/etu_day1.png)

Elis Hirwing (etu) talked about the Nix ecosystem! This was a great overview of
the different Nix components and tools.

Some take-aways from this talk:

- The [Nix pkgs](https://github.com/NixOS/nixpkgs) repository on Github is
  **huge**, over 49 000 packages! So this is a very active community. According
  to [Repology](https://repology.org/repositories/statistics/newest) it's the
  most up-to-date repo right now!
- The community works to keep packages as up to date as possible, and it is
  relatively easy to become a contributor.
- They try to remove unmaintained or EOL packages (unless too many other
  packages depend on it....looking at you Python 2!).
- You don't have to use NixOS to take advantage of Nix packages, they can be
  used on basically any Linux or Darwin (macOS) distribution.

![Elis Hirwig presentation
slide](/images/nixos_minicon_2020/etu_presentation.png)

With tools like [direnv](https://direnv.net/) and nix-shell, Nix is also great
for setting up development environments. There is also a lot of tooling for
different languages. This slide is an example of how Etu uses nix-shell to
get the dependencies needed for generating the slides of this presentation.
Nix has grown a lot in the last five years, and it is pretty exciting to
follow that development further down the road.

[Watch the talk](https://www.youtube.com/watch?v=9Su89RLoh0Q)

The slides are available on
[Github](https://github.com/etu/presentations/tree/master/oslo-nixos-meetup-2020-02-22)

### NixOps

![Kim Lindberger](/images/nixos_minicon_2020/talyz_day1.png)

Then Kim Lindberger (talyz) gave a great presentation on
[NixOps](https://nixos.org/nixops/). We even got treated to some demos!

Some things to note about NixOps:

- NixOps can be used to deploy NixOS systems to machines and non-machine
  resources (DNS, S3 buckets, etc.). All configuration is build locally before
  being shipped.
- There are plugins for a few cloud providers, for instance Amazon EC2 and
  Google Cloud Engine.
- If a deploy fails for some reason, you are never stuck with a system that is
  in a half-updated state. If the config doesn't build, it won't get pushed
  upstream and applied at all.
- NixOps is unfortunately still Python 2, but there are efforts on the way to
  port it to a modern Python.
- Backends will be split into separate plugins in an upcoming release!

![Kim Lindberger presentation
slide](/images/nixos_minicon_2020/talyz_presentation.png)

[Watch the talk](https://www.youtube.com/watch?v=SoHtccHNOJ8)

You can find the slides and the examples used in the demos on
[Github](https://github.com/talyz/presentations/tree/master/nixops-oslo-2020)

### Nix expressions

![Adam Höse](/images/nixos_minicon_2020/adis_day1.png)

Last talk of the day was Adam Höse (Adisbladis) giving us an intro to reading
Nix expressions. This is perhaps the most daunting aspect of NixOS for
beginners.

A few things to consider:

- It's not an imperative language, it is functional! A description that was
  mentioned is  "a little bit like a weird Lisp without all the parents"
- Using the [nix repl](https://nixos.wiki/wiki/Nix-repl) can be useful if you
  want to debug expressions or just play around with the language.
- Nix configurations can have different weights. Meaning that if you duplicate
  expressions, you can assign a weight to one of them that determines what will
  actually be built. Nix will merge all config together and that way decide what
  takes precedence.
- The key take-away: Learning the language will take your Nix journey further!

![Adam Höse presentation](/images/nixos_minicon_2020/adis_presentation.png)

Lots of questions were asked by the audience during this talk, and hopefully
some light was shed on the mysteries of the Nix language by the end.

[Watch the talk](https://www.youtube.com/watch?v=tJHb8Y_LOjE)

Then we ate some pizza and hung out Hackeriet style!  ![Waiting for pizza to
arrive](/images/nixos_minicon_2020/nix_hangout.jpg)

## DAY 2

### Building Docker containers with Nix

![Adam Höse](/images/nixos_minicon_2020/adis_day2.jpg)

On the last day we got an overview of how to build Docker containers using Nix
by Adam Höse.

- Normally a Docker build is an imperative process that might have different
  outcomes at different points in time (mostly because of the use of base
  images). Building a Docker image "Nix style" makes it reproducible, it will
  build the same way every time.
- Docker layers are content-addressed paths that are ordered; they are not
  sequential.
- `buildLayeredImage` is great for minimizing the amount of dependencies that
  are pulled into the final Docker image.
- [Nixery](https://nixery.dev/) is a project where you can get ad-hoc Docker
  images with a specific set of packages. The way to do this is to put the
  package names in the image name, like this: `docker pull
  nixery.dev/shell/git/htop`. Then you will get a custom docker image with bash,
  git and htop. Really cool!
  
![Day 2 of the con](/images/nixos_minicon_2020/day2_overview.jpg)

[Watch the talk](https://www.youtube.com/watch?v=9mcfadAoie8)

After this talk we had a informal session of hacking on Nix things and
socializing.

The organizers (fnords & sgo) want to thank the speakers and everyone else that
came to the mini-con! If you want to get notified of any future events in Oslo
NixOS User Group, you can join our [Meetup
group](https://www.meetup.com/Oslo-NixOS-User-Group/).
