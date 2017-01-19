Android Open Source Project Docker Build Environment
====================================================

Minimal build environment for AOSP with handy automation wrapper scripts.

Developers can use the Docker image to build directly while running the
distribution of choice, without having to worry about breaking the AOSP build
due to package updates as is sometimes common on rolling distributions like
Arch Linux.

Production build servers and integration test servers should also use the same
Docker image and environment. This eliminate most surprises in breakages by
by empowering developers and production builds to use the exact same
environment.  The hope is that breakages will be caught earlier by the devs.

This only works (well) on Linux.  Running this via `boot2docker` will result in
a very painful performacne hit due to VirtualBox's `vboxsf` shared folder
service which works terrible for **very** large builds like AOSP.  It might
work, but consider youself warned.  If you're aware of another way to get
around this, send a pull request!

For *Mac OS X* and *Windows* users, consider
[kylemanna/vagrant-aosp](https://github.com/kylemanna/vagrant-aosp) as a good
virtual machine to enable development.

Build Dockers
-------------

* For Nougat
`docker build -t nougat -f Dockerfile_7.0 .`
* For Marshmallow and Lollipop
`docker build -t lollipop -f Dockerfile_5.0-6.0 .`
* For kitkat
`docker build -t kitkat -f Dockerfile_kitkat .`


Quickstart
----------

For the terribly impatient.

1. Make a directory to work and go there.
2. Export the current directory as the persistent file store for the `aosp`
   wrapper.
3. Run a self contained build script, which does:
    1. Attempts to fetch the `aosp` wrapper if not found locally.
    2. Runs the `aosp` wrapper with an extra argument for the docker binary and
       hints to the same script that when run later it's running in the docker
       container.
    3. The aosp wrapper then does it's magic which consists of fetching the
       docker image if not found and forms all the necessary docker run
       arguments seamlessly.
    4. The docker container runs the other half the build script which
       initializes the repo, fetches all source code, and builds.
    5. In parallel you are expected to be drinking because I save you some time.


    mkdir kitkat ; cd kitkat
    export AOSP_VOL=$PWD
    curl -O https://raw.githubusercontent.com/kylemanna/docker-aosp/master/tests/build-kitkat.sh
    bash ./build-kitkat.sh

How it Works
------------

The Dockerfile contains the minimal packages necessary to build Android based
on the main Ubuntu base image.

The `aosp` wrapper is a simple wrapper to simplify invocation of the Docker
image.  The wrapper ensures that a volume mount is accessible and has valid
permissions for the `aosp` user in the Docker image (this unfortunately
requires sudo).  It also forwards an ssh-agent in to the Docker container
so that private git repositories can be accessed if needed.

The intention is to use `aosp` to prefix all commands one would run in the
Docker container.  For example to run `repo sync` in the Docker container:

    aosp repo sync -j2

The `aosp` wrapper doesn't work well with setting up environments, but with
some bash magic, this can be side stepped with short little scripts.  See
`tests/build-kitkat.sh` for an example of a complete fetch and build of AOSP.


Tested
------

* Android Kitkat `android-4.4.4_r2.0.1`
