---
date: 2018-02-11 12:00:03
title: Setting up Bors and Drone behind Nginx and SSL
---

I've recently found myself in the need of CI in a private Github repository.
Since I found Travis CI and other CI solutions to be way too expensive for what
I needed, I turned to open source solutions:

- [**Drone**][drone] is a staightforward and easy to use CI solution.
- [**Bors**][bors] is a merge automation tool, allowing you to have an evergreen
master.

Together they will sit behind inside docker containers and get reverse proxied
by an nginx instance of your choosing.

## Setting up the apps

- Bors requires a GitHub App, instructions on how to [create that are here][create_bors_app].
- Drone requires an OAuth App, it explains how you [configure that here][create_drone_app]
    (the Registration part).

Now that these exist, be sure to keep the pages open so you have the

##



[drone]: https://github.com/drone/drone
[bors]: https://github.com/bors-ng/bors-ng
[create_bors_app]: https://github.com/bors-ng/bors-ng#how-to-set-up-your-own-real-instance
[create_drone_app]: http://docs.drone.io/install-for-github/
