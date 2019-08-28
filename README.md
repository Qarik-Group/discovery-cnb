# Discovery buildpack

Buildpack that prints out arguments being passed between `bin/detect`, `bin/build`, and the resulting Docker image.

* https://buildpacks.io/docs/create-buildpack/building-blocks-cnb/
* https://github.com/buildpack/spec/blob/master/buildpack.md

**NOTE:** This buildpack was created with `pack 0.3.0` during August 2019. If the outputs or behavior changes, please let me know and I'll update the README or buildpack scripts.

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
