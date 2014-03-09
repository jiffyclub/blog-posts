[Docker][] is a great tool for getting lightweight, isolated Linux
environments. It uses technology that doesn't work natively on Macs.
Until now you've had to boot into a VM to install and use Docker,
but it's now a little easier than that.

As of Docker 0.8 it can be run on Macs thanks to a specially developed,
lightweight [VirtualBox][] VM. There are [official instructions][docker-mac]
for installing Docker on Mac, but with [Homebrew][brew] and [cask][] it's
even easier.

Follow the instructions on the [cask homepage][cask] to install it.
Cask is an extension to Homebrew for installing Mac binary packages via
the command line. Think things like Chrome or Steam. Or VirtualBox.
Running Docker on Mac requires VirtualBox so if you don't have it already:

    brew cask install virtualbox

Then install Docker and the helper tool `boot2docker`:

    brew install docker
    brew install boot2docker

`boot2docker` takes care of the VM that Docker runs in. To get things
started it needs to download the Docker VM and start a daemon that the
`docker` command line tool will talk to:

    boot2docker init
    boot2docker up

The `docker` command line tool should now be able to talk to the daemon
and if you run `docker version` you should see a report for both a server
and a client. (Note: When I ran `boot2docker up` it told me that the default
port the daemon uses was already taken. I had to specify a different port
via the `DOCKER_HOST` environment variable, which I now set in my shell
configuration.)

If everything has gone well to this point you should now be able to start
up a Docker instance. This command will drop you into a bash shell in Ubuntu:

    docker run -i -t ubuntu /bin/bash

Use `ctrl-D` to exit. I find this especially helpful for very quickly getting
to a Linux command line from my Mac for testing this or that, like checking
what versions of software are installing by `apt-get`.

Visit the [Docker documentation][docker-docs] to learn more about what you
can do with Docker and how to do it.

[Docker]: https://www.docker.io/
[VirtualBox]: https://www.virtualbox.org/
[docker-mac]: http://docs.docker.io/en/latest/installation/mac/
[brew]: http://brew.sh/
[cask]: http://caskroom.io/
[docker-docs]: http://docs.docker.io/en/latest/
