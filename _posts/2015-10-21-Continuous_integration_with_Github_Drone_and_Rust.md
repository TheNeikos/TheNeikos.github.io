---
layout: post
date: 2015-10-21 11:25:33
title: Continuous Integration with Github, Drone and Rust
---

Continuous Integration with Github, Drone and Rust
==================================================

The combination of Github and Drone (or any other CI) is quite powerful when it
comes to assuring that your pull requests are not breaking any existing tests.

In this Blog post I will show you how to get from 0 to a github repository with
tests for a rust library.

**The Result we want to have:**

1. Protected Master Branch
2. Tests are automatically run for all pull requests
3. Pull Requests have to pass all tests

**Pre-requisites**

- A working Ubuntu 14.04 Server with the latest updates
- A basic understanding of the Linux shell Bash
- A basic understanding of Docker
- A basic understanding of git
- A Github account

Getting Started
---------------

### Drone
First we will install drone on the Ubuntu server, so log in there.

    clouduser@drone-test:~$

Alright! Now let's get started:

    clouduser@drone-test:~$ sudo apt-get update
    / ..things.. /
    clouduser@drone-test:~$ sudo apt-get install docker.io

Once that is done we can install drone itself!


    clouduser@drone-test:~$ wget downloads.drone.io/master/drone.deb
    --2015-10-21 11:36:58--  http://downloads.drone.io/master/drone.deb
    Resolving downloads.drone.io (downloads.drone.io)... 54.231.32.52
    Connecting to downloads.drone.io (downloads.drone.io)|54.231.32.52|:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 7722384 (7.4M) [application/x-debian-package]
    Saving to: ‘drone.deb’

    100%[==============================================================================================>] 7,722,384   5.49MB/s   in 1.3s

    2015-10-21 11:37:00 (5.49 MB/s) - ‘drone.deb’ saved [7722384/7722384]

    clouduser@drone-test:~$ sudo dpkg --install drone.deb
    sudo: unable to resolve host drone-test
    Selecting previously unselected package drone.
    (Reading database ... 55631 files and directories currently installed.)
    Preparing to unpack drone.deb ...
    Unpacking drone (0.3.0-alpha-1442513246) ...
    Setting up drone (0.3.0-alpha-1442513246) ...
    Your system ubuntu 14: using upstart to control Drone
    drone start/running, process 8024

Now go checkout `http://yourip/`. It should display a link to the setup guide.
If you wish to also have SSL enable you should check it out, but otherwise this
post will also tell you the next steps.

Do not close the connection yet as we will have to fill the config in the next
step.

### Connecting Github to Drone

We now have to create a new Developer Application in Github, thankfully this is
rather easy. Go to: https://github.com/settings/developers and click on
**"Register new application"** at the top right.

Now fill out the fields:

![](https://i.imgur.com/bDpQQM9.png)

There you will find two important fields:

- **Client ID**
- **Client Secret**

Go back to your drone server and open `/etc/drone/drone.toml` as root with your
favorite editor and look for `[github]` and uncomment these lines:

    [github]
    client=""
    secret=""

Now you can fill those fields with the information from your github. Then save
and you can close the file. Now we restart the drone server so that it reads
the new configuration. You should now be greeted with a Github icon, allowing
you to sign in!

We now have a drone server up and running and we have it linked with Github,
now let's create a Rust library that we can test!

### Rust Library

If you still have the drone server open you may now close the connection.

If you have a place where you usually create projects go there now and then
let's create a new cargo lib.

    cargo new awesome_lib

We then create the first commit inside the new lib:

    cd awesome_lib
    neikos@celestias-palace ~/p/r/awesome_lib (master)> git add .
    neikos@celestias-palace ~/p/r/awesome_lib (master)> git commit -m "Initial Commit"
    [master (root-commit) a0b044a] Initial Commit
     3 files changed, 9 insertions(+)
     create mode 100644 .gitignore
     create mode 100644 Cargo.toml
     create mode 100644 src/lib.rs


Now let's create the repo. Let's go to https://github.com/new and create the
repo.

![](https://i.imgur.com/owN2yXK.png)

Once that is done we will tell our local repository that there is an upstream
repository. (Basically the 'push an existing repository')
Before copying the commands be sure to use ssh so that you authenticate through
that instead of typing your username/password everytime. (It can get annoying
quickly.)

    git remote add origin git@github.com:TheNeikos/awesome_lib.git
    git push origin master

Awesome! We now get to the interesting parts.

### Adding the repository to drone

Navigate to the drone instance you have setup earlier in your browser, login and
press the 'sync' button at the top right. This is needed in case Drone does not
pick up on the repository right away. Click on 'Browse' and look for
'awesome\_lib', click on it and then press the "Activate Now" button.

Once loading process is done let's create the `.drone.yml` file so that the
server actually knows what's up!


### Creating the .drone.yml file

In your local awesome\_lib directory create a .drone.yml

Be sure to replace your docker name correctly, feel also free to update the rust
version should yours differ (it probably will in the future)

If you don't have a docker account yet you can create one
[here](https://docs.docker.com/docker-hub/accounts/)

```yml
image: <your docker name>/rust:1.3
env:
    - CARGO_TARGET_DIR=/var/cache/drone/cargo
    - CARGO_HOME=/var/cache/drone/cargo
script:
    - cargo build
    - cargo test
cache:
    - /var/cache/drone/cargo
```

Then we create a new branch and add our changes to that and push them!

    neikos@celestias-palace ~/p/r/awesome_lib (master)> git checkout -b add-drone
    neikos@celestias-palace ~/p/r/awesome_lib (add-drone)> git add .drone.yml
    neikos@celestias-palace ~/p/r/awesome_lib (add-drone)> git commit -m "Add .drone.yml"
    [add-drone b894e05] Add .drone.yml
     1 file changed, 9 insertions(+)
     create mode 100644 .drone.yml
    neikos@celestias-palace ~/p/r/awesome_lib (add-drone)> git push origin add-drone
    Counting objects: 3, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (3/3), done.
    Writing objects: 100% (3/3), 1.08 KiB | 0 bytes/s, done.
    Total 3 (delta 0), reused 0 (delta 0)
    To git@github.com:TheNeikos/awesome_lib.git
     * [new branch]      add-drone -> add-drone

Perfect. Now let's make that pull request! If you go back to your repo on Github
it should tell you that you can create a Pull Request.

Afterwards it should load and quickly tell you that it has failed, the reason is
simply that the image does not exist yet, let's do that now!

### Creating the Docker Image

Okay, so let's create a dockerfile that will allow us to have cargo and rust.

```text
FROM ubuntu:15.04

RUN apt-get install curl file git build-essential -y && curl -sSf https://static.rust-lang.org/rustup.sh \
    | sh -s -- -y --disable-sudo
```

Then we build it!

    neikos@celestias-palace ~/t/rust_image> docker build .

It should finish with:

        Rust is ready to roll.

     ---> bc9ad00de680
    Removing intermediate container 040774e66535
    Successfully built bc9ad00de680

In that last line is an id, we will need that in the next step, copy it and
put it in the next command:

    neikos@celestias-palace ~/t/rust_image> docker tag <THE_ID> <DOCKER_USERNAME>/rust:1.3

Then we upload it:

    neikos@celestias-palace ~/t/rust_image> docker push neikos/rust:1.3
    The push refers to a repository [docker.io/neikos/rust] (len: 1)
    bc9ad00de680: Image successfully pushed
    76be1ef20317: Image already exists
    82554298ff4f: Image already exists
    08ab09376d9a: Image already exists
    9f5beeea5d8a: Image already exists
    b8b73eaafc6e: Image already exists
    1.3: digest: sha256:73ab1f422b220511286e6b7afcdd1decaaf1081fcb3d6b68c758b7a99dd4cdf9 size: 10109

Now we restart the drone build. Click on the red cross next to your commit
in your pull request on Github. It should take you to your drone server. And if
you are logged in, you should be able to click on 'rebuild' to kick off the
build again.

The first time this runs might take a bit longer as the image will have to be
downloaded first, subsequent builds should be faster. On my virtual machine it
takes about 1-2 minutes.

![](https://i.imgur.com/cET7COE.png)

At the end it should tell you that it passed, you can then go back to you Github
and start using it! Or secure it a bit more against accidental merges.

### Forcing checks before merge

Navigate to your repository and click on settings on the bottom left, then go
to the branches tab and protect the `master` branch.

Then you can re-affirm that you want to protect the branch and require status
checks before merging. I like to include administrators because they can
write code that is just as bad :) as the average person.

Don't forget to check Drone at the bottom to make it **required**.


And that's it! Feel free to start writing tests and make pull requests! From now
on they will always be checked and you shouldn't have any failing tests in your
master branch, isn't that awesome!

If you need any additional dependencies you can add them to the dockerfile and
update the image.
