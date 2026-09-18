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

The container files layer on top of one another so the base stays
consistent across environments:  a shared base, then an environment
layer (language and tooling), then a harness layer for AI development.
Every image runs a firewall configuration script on startup as root
using a **sudoer** rule that allows the default user, 'pineapple', to
do this, and then runs the pi coding agent.

1. **pineapple-base-container**  —  the shared base:  OS packages and
   tools, the `pineapple` user, and the global pi coding agent
   install.  Built first.

2. Environment and harness layers, each depending on
   `pineapple-base`:

   - **pineapple-clojure-tools-container**  —  the Clojure
     environment:  a JDK (Temurin 25) and maven, the Clojure CLI,
     clojure-lsp, clj-kondo, babashka and cljfmt, plus the
     `pi-clojure` package so any agent in this image can interact
     with the Clojure development environment.

   - **pineapple-mtplx-base-container** and
     **pineapple-omlx-base-container**  —  standalone harness
     images that install their pi packages (`pi-mtplx` and
     `pi-local-models` respectively) without the Clojure tooling.
     The mtplx image also sets up auth and model entries for the
     mtplx model server; the server host is the `MTPLX_HOST` build
     argument (default `192.168.64.1`).

3. Harness-specific Clojure images:  **pineapple-mtplx-clojure-container**
   and **pineapple-omlx-clojure-container** depend on
   `pineapple-clojure-tools` and layer in their harness-specific pi
   extension and configuration.  Each also refreshes the pi install
   on build, so rebuilding a leaf image picks up the latest pi
   without rebuilding the earlier layers.  Adding a third harness is
   a matter of writing another thin container file that depends on
   `pineapple-clojure-tools`.

The dependency tree:

   ```text
   node:26-trixie-slim
     └─ pineapple-base
          ├─ pineapple-clojure-tools
          │    ├─ pineapple-mtplx-clojure
          │    └─ pineapple-omlx-clojure
          ├─ pineapple-mtplx-base          (standalone)
          └─ pineapple-omlx-base           (standalone)
   ```

To build a Clojure image, build up its lineage in order, e.g.

   ```text
   pineapple-build -f pineapple-base-container
   pineapple-build -f pineapple-clojure-tools-container
   pineapple-build -f pineapple-mtplx-clojure-container
   ```

The standalone harness images build straight off the base, e.g.
`pineapple-build -f pineapple-mtplx-base-container`.

## Credits
Pi agent harness: https://pi.dev/

Apple Containers: https://github.com/apple/container

Richard Towers: Claude Code in Apple Container

Michael Hannecke: Pi in Apple Container
