# heroku-buildpack-imagemagick-heif

The rise in popularity and use of HEIF/HEIC (High Efficiency Image Format) means your project's image processing also needs to be able to handle this format.
The default ImageMagick shipped in the Heroku stacks is a 6.x build from Ubuntu and does not support processing HEIC image files.

This [Heroku buildpack](https://devcenter.heroku.com/articles/buildpacks) vendors a version of ImageMagick 7 with **WEBP and HEIF support** into your project.

This fork targets [**heroku-24**](https://devcenter.heroku.com/articles/heroku-24-stack) (Ubuntu 24.04). It was rebuilt because the upstream yespark tarball linked against `libtiff.so.5` from Ubuntu 22.04, which Ubuntu 24.04 drops in favour of `libtiff.so.6`. Running that tarball on heroku-24 fails at runtime with `error while loading shared libraries: libtiff.so.5: cannot open shared object file`. Rebuilding inside `heroku/heroku:24-build` re-links against the stack's `libtiff6`.

The tar file in the [/build folder](./build) currently contains:

Version: ImageMagick 7.1.0-53 (linked against Ubuntu 24.04 system libraries)

You will need to build a new binary if you want to use a newer or different version, or target a different stack. See [How to Build a New Binary](#how-to-build-a-new-binary).

# Usage

## Step 1 : Adding the buildpack

From your projects "Settings" tab add this buildpack to your app in the 1st position:

```bash
https://github.com/iPadReceptionist/heroku-imagemagick-buildpack
```

**NOTE:** \__To ensure the newer version of imagemagick is found in the $PATH and installed first make sure this buildpack is added to the top of the buildpack list or at "index 1"._

## Step 2 : Clear the cache(**Not Sure if this is necessary**)

Since the installation is cached you might want to clean it out due to config changes.

```bash
heroku plugins:install heroku-builds
heroku builds:cache:purge -a HEROKU_APP_NAME
```

# How to Build a New Binary (if you want to make somes changes)

The binary in this repo was built in a heroku:22 docker image running in a local dev environment.
However, there is a script called [**build.sh**](./build.sh) made to build a tar file through docker easily, it will be copied to the `build` directory. Then you should commit this changes to your git, and adjust the buildpack url previously mentionned just above.

The binary currently in this repo was built inside a `heroku/heroku:24-build` Docker image. To retarget another stack, change the `FROM` line in [`container/Dockerfile`](./container/Dockerfile) to the matching Heroku build image (e.g. `heroku/heroku:22-build` for heroku-22) and rerun `./build.sh`.

## Prerequisites

- Docker installed and running in local dev environment. [Get Docker](https://docs.docker.com/get-docker/)

## Credits

This buildpack is a fork of [yespark/heroku-imagemagick-buildpack](https://github.com/yespark/heroku-imagemagick-buildpack), maintained by The Receptionist.

Other references:

- https://medium.com/@eplt/5-minutes-to-install-imagemagick-with-heic-support-on-ubuntu-18-04-digitalocean-fe2d09dcef1
- https://github.com/brandoncc/heroku-buildpack-vips
- https://github.com/steeple-dev/heroku-buildpack-imagemagick
- https://github.com/retailzipline/heroku-buildpack-imagemagick-heif

## License

The gem is available as open source under the terms of the [MIT License](https://github.com/iPadReceptionist/heroku-imagemagick-buildpack/blob/master/LICENSE).
