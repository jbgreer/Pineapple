# pineapple:  Pi agent harness in an Apple Container
An example running [Pi agent harness](https://pi.dev) inside an [Apple Container](https://github.com/apple/container).

An uncommon feature: this sets firewall rules inside the container.
This used to 'just work', but now requires setting a capability.

This relies on project-level configure and specifically does not 
mount the host .pi direct as a configuration directory.  Curiously,
out-of-the-box pi does not support a project level model.json.
I'm relying on pi-local-models package.
For the included Clojure example I have switched from clojure-mcp-light
to the pi-clojure package, just cause I wanted to - don't read anything into it.

## Prerequisites

[Apple Container](https://github.com/apple/container)

## Steps

1) Start the `apple/container` runtime

  ```text
  container system start`
  ```

2) Optionally, copy the `pineapple-build` and `pineapple-run` 
   helper script onto your PATH, e.g.

   ```shell
   cp pineapple-build /usr/local/bin/
   cp pineapple-run /usr/local/bin/
   ```

   The helper scripts default to building and running the image defined
   in the `pineapple-base-container`.

3) Build a container image

   ```text
   container build -f pineapple-base-container -t pineapple-base
   ```
   OR use the helper script.  The tag is optional and
   defaults to the filename with the `-container`
   suffix stripped:

   ```text
   pineapple-build
   pineapple-build -t my-tag
   ```

   Note this builds the first tier only; see [Container files](#container-files)
   below for the full build order.

4) Run the container from a project directory. 
   You can either use the helper script or run the command directly.
   Note that the container file invokes a script to set firewall
   permissions and requires the CAP_NET_ADMIN capability.

   ```text
   container run \
     --cap-add CAP_NET_ADMIN \
     -c {cpus} -m {mem} \
     -i -t \
     -v {hwd} \
     {image}
   ```

   See `container` documentation for options.

   OR use the helper script
   ```text
   pineapple-run
   ```

## Container files

The container images are built in three tiers, each tier depending on
the one below it.  Every tier runs a firewall configuration script on
startup as root using a **sudoer** rule that allows the default user,
'pineapple', to do this, and then runs the pi coding agent.

1. **pineapple-base-container**  —  the shared base:  OS packages and
   tools, the `pineapple` user, and the global pi coding agent
   install.  Built first.

2. **pineapple-mtplx-base-container** and
   **pineapple-omlx-base-container**  —  depend on
   `pineapple-base` and install their pi packages
   (`pi-mtplx` and `pi-local-models` respectively).  The mtplx base
   also sets up auth and model entries for the mtplx model server;
   the server host is the `MTPLX_HOST` build argument
   (default `192.168.64.1`).

3. **pineapple-mtplx-clojure-container** and
   **pineapple-omlx-clojure-container**  —  depend on their matching
   `-base-container` image and additionally install Clojure and a
   number of other useful programs and utilities.

To build, build up the tiers in order, e.g.

   ```text
   pineapple-build -f pineapple-base-container
   pineapple-build -f pineapple-mtplx-base-container
   pineapple-build -f pineapple-mtplx-clojure-container
   ```

## Credits
Pi agent harness: https://pi.dev/

Apple Containers: https://github.com/apple/container

Richard Towers: Claude Code in Apple Container

Michael Hannecke: Pi in Apple Container
