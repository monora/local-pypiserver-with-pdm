# Use pypi-server with PDM for local packages

## Installation

See [pypiserver](https://pypi.org/project/pypiserver/#using-the-docker-image) and local `docker-compose.yml`

```bash
docker-compose up -d
```

[Simple Index](http://localhost:8082/simple/) shows all locally deployed packages.


## Configuration

### Authentication

Since we up and download only packages on the same machine we configure
unauthenticated access `-a . -P .` in `docker-compose.yml`:

```yaml
pypiserver:
# No authentication!
  command: run -a . -P .
```

### PDM add dependency (download)

#### Configure local repository as additional repo

See [Specify other sources for finding packages](https://pdm.fming.dev/latest/pyproject/tool-pdm/#specify-other-sources-for-finding-packages) in PDM documentation.

```toml
[[tool.pdm.source]]
url = "http://localhost:8082"
name = "local"
```

To add a dependency to a package in our local pypi repository we can simply add it as normal dependency in PDM:

```bash
pdm add <pypackage>
```

### PDM publish (upload)

#### pdm/config.toml

Also not needed username and password must be defined in `~/.config/pdm/config.toml` to avoid errors when publishing.

```bash
pdm config --global repository.local.url "http://localhost:8082"
pdm config --global repository.local.username "doesnotmatter"
pdm config --global repository.local.url "doesnotmatter"
```

The repository to publish to can be specified as environment variable like so:

```bash
export PDM_PUBLISH_REPO=local
```


Now the local package can be [published](https://pdm.fming.dev/latest/usage/project/#publish-the-project-to-pypi) with PDM.

```bash
pdm publish -v
```

#### Versioning

See [Dynamic versioning](https://pdm.fming.dev/latest/pyproject/build/#dynamic-versioning) with this configuration in `pyproject.toml`:

```toml
[tool.pdm.version]
source = "scm"
write_to = "<module_name>/__version__.py"
write_template = "__version__ = '{}'"
```

The version can be specified as environment variable like so:

```bash
export PDM_PEP517_SCM_VERSION="0.1.0"
```
