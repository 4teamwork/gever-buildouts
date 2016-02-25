# GEVER Buildouts

This repository provides standard buildouts which can be extended in order to
deploy a [GEVER](https://github.com/4teamwork/opengever.core).

It bases on and extends
[ftw-buildouts](https://github.com/4teamwork/ftw-buildouts).

This repository does not contain actual deployment buildouts but only components
which can be put together to have buildout.
Actual deployment buildouts should be placed in the project repository.


## Deployment buildout

For creating a buildout config for deploying a GEVER, you can include various
configuration blocks from `gever-buildouts` via HTTP buildout extends in order
to have certain features.

Each configuration file may require you to set certain configuration values,
usually in buildout variables of the main `[buildout]` section.

### Required blocks

For a GEVER to work, you must include certain configuration blocks by extending
the buildouts, others are optional.
These extends are required:

- ``standard-deployment.cfg``: Contains the basic deployment configuration.


## [standard-deployment.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/standard-deployment.cfg)

This is the main deployment buildout which should be extended by all
deployments.
It installs the default eggs and includes the standard deployment components,
such as:

- supervisor setup with a ZEO client / server setup, by including
  `ftw-buildout`'s
  [production.cfg](https://github.com/4teamwork/ftw-buildouts/blob/master/production.cfg)
- standard Sablon setup with ruby configuration
- warmup: automatically warmup instances after start
- `standard-sources.cfg` (see below)
- standard eggs such as `opengever.maintenance` and `ftw.zopemaster`

**Variables:**
- `buildout:deployment-number`: deployment number, prefixing all ports
- `buildout:client-policy`: dotted name to the client policy; its ZCML will be
  loaded automatically

**Example:**

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
deployment-number = 32
ogds-db-name = onegovgever-buildoutdemo
```
