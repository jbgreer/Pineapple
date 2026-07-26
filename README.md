# pineapple:  pi coding agent in an Apple Container
An example running [pi coding agent](https://pi.dev) inside an [Apple Container](https://github.com/apple/container).

## Prerequisites

- Apple Container.

## Steps

1) Make sure the `apple/container` runtime is running with `container system start`


2) Build the container image

   ```text
   container build --memory 4g --tag pineapple --file .devcontainer/Containerfile
   ```


3) Optionally, copy the `pineapple` helper script onto your PATH, e.g.

   ```shell
   cp pineapple /usr/local/bin/
   ```

4) From your project directory, run the container. You can either use the
   helper script or run the command directly:

   **Using the helper script** — get an interactive zsh shell:

   ```text
   pineapple
   ```
   **Or, run the command directly:**

   ```text
   container run --name pineapple \
     --internal \
     --memory 8g \
     --volume "$(pwd):/workspace" \
     --detach --rm pineapple sleep infinity
   ```
   The named volumes persist shell history and pi configuration across
   container restarts.


## Credits
[pi coding agent]: https://pi.dev/
[apple/container]: https://github.com/apple/container
[Richard Towers]: Claude Code in Apple Container
[Michael Hannecke] Pi in Apple Container
