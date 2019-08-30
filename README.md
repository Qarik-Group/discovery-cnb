# Discovery buildpack

Buildpack that prints out arguments being passed between `bin/detect`, `bin/build`, and the resulting Docker image.

* https://buildpacks.io/docs/create-buildpack/building-blocks-cnb/
* https://github.com/buildpack/spec/blob/master/buildpack.md

**NOTE:** This buildpack was created with `pack` from source, just prior to v0.4.0 release during August 2019. If the outputs or behavior changes, please let me know and I'll update the README or buildpack scripts.

**NOTE:** The `builder.toml` explicitly uses [`lifecycle` 0.4.0](https://github.com/buildpack/lifecycle/releases/tag/v0.4.0) (`pack` 0.3.0 defaults to `lifecycle` 0.3.0)

## Playtime with buildpack

```plain
pack build discovery-app --buildpack . --path fixtures/ruby-sample-app
```

To see inside the runtime Docker image:

```plain
docker run -ti discovery-app bash
```

The original `fixtures/ruby-sample-app` is placed in the working directory at `/workspace`:

```plain
$ pwd
/workspace
$ ls
Gemfile  Gemfile.lock  app.rb  vendor
```

A metadata file is created from the buildpacks detected and used to create the application:

```plain
$ cat /layers/config/metadata.toml
processes = []
buildpacks = ["com.starkandwayne.buildpacks.discovery"]

[[processes]]
  type = "web"
  command = "nc-webapp"

[bom]
  [bom.discovery]
    message = "hello"
    version = "1.2.3"
```

The layers added by the buildpacks are at `/layers`:

```plain
$ ls -alR /layers/com*/
/layers/com.starkandwayne.buildpacks.discovery/:
total 12
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 1 vcap vcap 4096 Jan  1  1980 ..
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 discovery-launch-true

/layers/com.starkandwayne.buildpacks.discovery/discovery-launch-true:
total 12
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 ..
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 bin
-rw-r--r-- 1 vcap vcap    0 Jan  1  1980 some-file

/layers/com.starkandwayne.buildpacks.discovery/discovery-launch-true/bin:
total 16
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 ..
-rwxr-xr-x 1 vcap vcap   32 Jan  1  1980 hello
-rwxr-xr-x 1 vcap vcap   27 Jan  1  1980 nc-webapp
```

The `hello` and `nc-webapp` executables are automatically added into the `$PATH`:

```plain
$ hello
hello world

$ echo $PATH
/layers/com.starkandwayne.buildpacks.discovery/discovery-launch-true/bin:...
```

Run the tiny `nc`-based web app explicitly:

```plain
$ docker run -ti -p 8080:8080 --env PORT=8080 discovery-app nc-webapp
Listening on [0.0.0.0] (family 0, port 8080)
```

But the `nc-webapp` is also described by `bin/build` as a launch process, so it is automatically run as the default behavior of the Docker image:

```plain
$ docker run -ti -p 8080:8080 --env PORT=8080 discovery-app
Listening on [0.0.0.0] (family 0, port 8080)
```

## Playtime with builder containing buildpack

```plain
pack create-builder starkandwayne/discovery-cnb -b builder.toml
```

The output may look like:

```plain
Creating builder starkandwayne/discovery-builder from build-image cloudfoundry/cnb-build:cflinuxfs3
Downloading from "https://github.com/buildpack/lifecycle/releases/download/v0.3.0/lifecycle-v0.3.0+linux.x86-64.tgz"
Successfully created builder image starkandwayne/discovery-builder
Tip: Run pack build <image-name> --builder starkandwayne/discovery-builder to use this builder
```

Now we can try applying the buildpack automatically from within our custom builder:

```plain
pack build discovery-app --builder starkandwayne/discovery-builder --path fixtures/ruby-sample-app
```

The output shows that our buildpack is successfully detected and built into the final application Docker image:

```plain
Selected run image cloudfoundry/cnb-run:cflinuxfs3
Pulling image cloudfoundry/cnb-run:cflinuxfs3
cflinuxfs3: Pulling from cloudfoundry/cnb-run
Digest: sha256:d5ac181ce8f1f3e9411a693d43b89508dc00bfa3509979dacf8c6ae00911f7cb
Status: Image is up to date for cloudfoundry/cnb-run:cflinuxfs3
Using build cache volume pack-cache-7296676022e3.build
Executing lifecycle version 0.3.0
===> DETECTING
[detector] Trying group 1 out of 1 with 1 buildpacks...
[detector] ======== Output: Discovery Buildpack ========
[detector] $1 platform: /platform
[detector] /platform/:
[detector] total 12
[detector] drwxr-xr-x 1 root root 4096 Aug 28 23:13 .
[detector] drwxr-xr-x 1 root root 4096 Aug 28 23:13 ..
[detector] drwxr-xr-x 1 root root 4096 Aug 28 23:13 env
[detector]
[detector] /platform/env:
[detector] total 8
[detector] drwxr-xr-x 1 root root 4096 Aug 28 23:13 .
[detector] drwxr-xr-x 1 root root 4096 Aug 28 23:13 ..
[detector]
[detector] $2 plan: /tmp/0.0.1.plan.966117460/plan.toml
[detector] -rwxr-xr-x 1 vcap vcap 53 Aug 28 23:13 /tmp/0.0.1.plan.966117460/plan.toml
[detector] ======== Results ========
[detector] pass: Discovery Buildpack
===> RESTORING
===> ANALYZING
[analyzer] Analyzing image '229b154bb00578763c0e4717d93b5089f6537412ae62b0ebab159bfd3580931d'
[analyzer] Writing metadata for uncached layer 'com.starkandwayne.buildpacks.discovery:discovery-launch-true'
===> BUILDING
[builder] ---> Discovery Buildpack
[builder] $1 layers: /layers/com.starkandwayne.buildpacks.discovery
[builder] /layers/com.starkandwayne.buildpacks.discovery:
[builder] total 12
[builder] drwxr-xr-x 2 vcap vcap 4096 Aug 28 23:13 .
[builder] drwxr-xr-x 3 vcap vcap 4096 Aug 28 23:13 ..
[builder] -rw-r--r-- 1 vcap vcap   42 Aug 28 23:13 discovery-launch-true.toml
[builder]
[builder] $2 platform: /platform
[builder] /platform:
[builder] total 12
[builder] drwxr-xr-x 1 root root 4096 Aug 28 23:13 .
[builder] drwxr-xr-x 1 root root 4096 Aug 28 23:13 ..
[builder] drwxr-xr-x 1 root root 4096 Aug 28 23:13 env
[builder]
[builder] /platform/env:
[builder] total 8
[builder] drwxr-xr-x 1 root root 4096 Aug 28 23:13 .
[builder] drwxr-xr-x 1 root root 4096 Aug 28 23:13 ..
[builder]
[builder] $3 plan: /tmp/plan.833650968/com.starkandwayne.buildpacks.discovery/plan.toml
[builder] -rwxr-xr-x 1 vcap vcap 0 Aug 28 23:13 /tmp/plan.833650968/com.starkandwayne.buildpacks.discovery/plan.toml
[builder]
[builder] STDIN:
[builder] [discovery]
[builder]   message = "hello"
[builder]   version = "1.2.3"
[builder] ---> Make some layers
[builder] ---> create bin/hello
[builder] ---> create tiny webapp with nc
===> EXPORTING
[exporter] Reusing layers from image with id '229b154bb00578763c0e4717d93b5089f6537412ae62b0ebab159bfd3580931d'
[exporter] Reusing layer 'app' with SHA sha256:4485fd00798ca2d1253b5ba46d4596acaf851d9d93de119efa7ce5de84243b06
[exporter] Reusing layer 'config' with SHA sha256:60c732e5c5769263b6f67fcd4530674301d425cbdd8a16b48b7b91851fbf5a66
[exporter] Reusing layer 'launcher' with SHA sha256:c8a8ddb80dd9923057bd12f9f69c6b093925a8925f3c37550a88b90f02699aa9
[exporter] Reusing layer 'com.starkandwayne.buildpacks.discovery:discovery-launch-true' with SHA sha256:1b1c0bd2e64912ea2731debd41c3bbaf2b918a0cb167b33a2ed4a99d8344aafc
[exporter] *** Images:
[exporter]       index.docker.io/library/discovery-app:latest - succeeded
[exporter]
[exporter] *** Image ID: 73e6e712829833ca914ed6685f86d253367e3945b0f0cbc7759b474f23d6e420
===> CACHING
Successfully built image discovery-app
```

## kpack

To install the `discovery-builder` builder into kpack, that contains this buildpack, use:

```plain
kubectl apply -f kpack-builder.yml
```

To apply the buildpack, via the builder, to an example project (this repo) apply the sample file. First update it for your own `spec.tag` and `spec.serviceAccount`.

```plain
kubectl apply -f kpack-fixture-app-build-from-git.yml
```

It will take a while for the first build to kick off, as Kubernetes needs to download the new `starkandwayne/discover-builder` image from Docker Hub registry.

To watch the buildpack lifecycle use kpack's `logs` CLI:

```plain
logs -image discovery-cnb-fixture-app
```

Run the published image locally:

```plain
docker run -ti -p 8080:8080 --env PORT=8080 starkandwayne/discovery-cnb-fixture-app
```

Subsequent changes to this Git repo should automatically trigger kpack to rebuild the Docker image.

