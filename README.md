# Discovery buildpack

Buildpack that prints out arguments being passed between `bin/detect`, `bin/build`, and the resulting Docker image.

* https://buildpacks.io/docs/create-buildpack/building-blocks-cnb/
* https://github.com/buildpack/spec/blob/master/buildpack.md

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
buildpacks = ["com.examples.buildpacks.discovery"]

[bom]
  [bom.discovery]
    message = "hello"
    version = "1.2.3"
```

The layers added by the buildpacks are at `/layers`:

```plain
$ ls -alR /layers/com*/
/layers/com.examples.buildpacks.discovery/:
total 12
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 1 vcap vcap 4096 Jan  1  1980 ..
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 discovery-launch-true

/layers/com.examples.buildpacks.discovery/discovery-launch-true:
total 12
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 ..
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 bin
-rw-r--r-- 1 vcap vcap    0 Jan  1  1980 some-file

/layers/com.examples.buildpacks.discovery/discovery-launch-true/bin:
total 12
drwxr-xr-x 2 vcap vcap 4096 Jan  1  1980 .
drwxr-xr-x 3 vcap vcap 4096 Jan  1  1980 ..
-rwxr-xr-x 1 vcap vcap   32 Jan  1  1980 hello
```

The `hello` executable is automatically added into the `$PATH`:

```plain
$ hello
hello world

$ echo $PATH
/layers/com.examples.buildpacks.discovery/discovery-launch-true/bin:...
```
