# pineapple:  pi coding agent in an Apple Container
An example running [pi coding agent](https://pi.dev) inside an [Apple Container](https://github.com/apple/container).

An uncommon feature: this sets firewall rules inside the container.
This used to 'just work', but not requires setting a capability.

## Prerequisites

[Apple Container](https://github.com/apple/container)

## Steps

1) Start the `apple/container` runtime

  ```text
  container system start`
  ```

2) Build a container image

   ```text
   container build --tag pineapple --file Pineapple-Containerfile
   ```

3) Optionally, copy the `pineapple` helper script onto your PATH, e.g.

   ```shell
   cp pineapple /usr/local/bin/
   ```

4) Run the container from a project directory. 
   You can either use the helper script or run the command directly.
   Note that the container file invokes a script to set firewall
   permissions and requires the CAP_NET_ADMIN capability.

   ```text
   pineapple
   ```
   **Or**

   ```text
   container run --name pineapple \
     --cap-add CAP_NET_ADMIN \
     --memory 2g \
     --volume "$(pwd):/workspace" \
     --detach --rm pineapple sleep infinity
   ```
## Container file

Both of the included container files run a firewall configuration script on 
startup as root.  There's a sudoer rule that allows the default user,'
pineapple, to do this.  They then both run the pi coding agent.

The **Pineapple-Clojure-Containerfile** differs in that it also installs
Clojure.

## Credits
[pi coding agent]: https://pi.dev/
[apple/container]: https://github.com/apple/container
[Richard Towers]: Claude Code in Apple Container.
[Michael Hannecke] Pi in Apple Container
