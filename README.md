# pineapple:  Pi agent harness in an Apple Container
An example running [Pi agent harness](https://pi.dev) inside an [Apple Container](https://github.com/apple/container).

An uncommon feature: this sets firewall rules inside the container.
This used to 'just work', but now requires setting a capability.

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
   in the `Pineapple-Base-Containerfile`.

3) Build a container image

   ```text
   container build -f Pineapple-Base-Containerfile -t pineapple-base
   ```
   OR use the helper script

   ```text
   pineapple-build
   ```

4) Run the container from a project directory. 
   You can either use the helper script or run the command directly.
   Note that the container file invokes a script to set firewall
   permissions and requires the CAP_NET_ADMIN capability.

   ```text
   container run \
     --cap-add CAP_NET_ADMIN \
     -c {cpus} -m {mem} \
     -i -t \
     -v {pcd} 
     -v {hwd} \
     {image}
   ```

   See `container` documentation for options.

   OR use the helper script
   ```text
   pineapple-run
   ```

## Container file

Both of the included container files run a firewall configuration script on 
startup as root using a **sudoer** rule that allows the default user,'
pineapple, to do this.  They then both run the pi coding agent.

The **Pineapple-Clojure-Containerfile** differs in that it also installs
Clojure and a number of other useful programs and utilities.  
To use it, you must first build the **pineapple-base** image.

## Credits
Pi agent harness: https://pi.dev/

Apple Containers: https://github.com/apple/container

Richard Towers: Claude Code in Apple Container

Michael Hannecke: Pi in Apple Container
