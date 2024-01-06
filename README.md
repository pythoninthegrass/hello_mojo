# hello_🔥

Goes through the official Mojo [example repo](https://github.com/modularml/mojo) with some sugar on top.

## Setup
### Minimum Requirements
* Get auth key from [dev console](https://developer.modular.com/download)
* Install CLI + SDK
    ```bash
    curl https://get.modular.com | sh - \
    && modular auth <AUTH_KEY>
    ```
* Install [Python](https://www.python.org/downloads/)
* Install [Jupyter](https://jupyter.org/install)

### Recommended Requirements
* [asdf](https://asdf-vm.com/#/core-manage-asdf-vm)
  * cf. the [mojo shim](bin/mojo)
* [docker](https://docs.docker.com/get-docker/)

## Quickstart
### Bare metal
```bash
# run script in interpreter
mojo hello.🔥

# compile script to binary
mojo build hello.🔥 -o bin/hello
```

### Docker
`TODO`

## Further Reading
[Modular Docs - Mojo🔥](https://docs.modular.com/mojo/)

[Modular Docs | Develop in the Mojo Playground](https://docs.modular.com/mojo/manual/get-started/index.html#develop-in-the-mojo-playground)

[Modular Discord](https://discord.gg/modular)
