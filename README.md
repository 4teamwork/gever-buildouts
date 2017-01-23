# GEVER Buildouts

This repository provides standard buildouts which can be extended in order to
deploy a [GEVER](https://github.com/4teamwork/opengever.core).

It bases on and extends
[ftw-buildouts](https://github.com/4teamwork/ftw-buildouts).

This repository does not contain actual deployment buildouts but only components
which can be put together to have buildout.
Actual deployment buildouts should be placed in the project repository.



## Deployment Buildout

For creating a buildout config for deploying a GEVER, you can include various
configuration blocks from `gever-buildouts` via HTTP buildout extends in order
to have certain features.

Each configuration file may require you to set certain configuration values,
usually in buildout variables of the main `[buildout]` section.



### Required Blocks

For a GEVER to work, you must include certain configuration blocks by extending
the buildouts, others are optional.
These extends are required:

- ``standard-deployment.cfg``: Contains the basic deployment configuration.
- Choose an **OGDS database**, such as:
  - ``ogds-postgres.cfg``
  - ``ogds-mysql.cfg``
- Amount of **ZEO-clients**:
  If you want more than one ZEO client you need to extend a client configuration
  from [ftw-buildout's `zeoclients` folder](https://github.com/4teamwork/ftw-buildouts/tree/master/zeoclients)
- A standalone or a shared **Tika**:
  - standalone: [tika-standalone.cfg](https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/tika-standalone.cfg)
  - shared: *to be implemented*
- Choose a GEVER version by extending the
  [**KGS**](http://kgs.4teamwork.ch/release/opengever/) - or you may use the
  `source-master.cfg` (see below)

**Extensions:**

- [Maintenance Server](https://github.com/4teamwork/ftw-buildouts#maintenance-http-server)



### Example Configuration

Assuming we have a policy package ``gever.hinterfultigen``, we would add a deployment
configuration directly in this policy package.
Lets name the buildout configuration
``deployment-artemis-19-hinterfultigen.onegovgever.ch``:

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/ogds-postgres.cfg
    https://raw.githubusercontent.com/4teamwork/ftw-buildouts/master/zeoclients/4.cfg
    https://raw.githubusercontent.com/4teamwork/ftw-buildouts/master/tika-jaxrs-server.cfg
    http://kgs.4teamwork.ch/release/opengever/3.6.0

deployment-number = 19
ogds-db-name = gever-hinterfultigen
client-policy = gever.hinterfultigen
usernamelogger_ac_cookie_name = __ac_hinterfultigen
instance-eggs += gever.hinterfultigen
raven_tags = {"cluster": "hinterfultigen.onegovgever.ch"}
develop = .
```



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
- `usernamelogger_ac_cookie_name`: (optional) `__ac` cookie name, defaults to
  `__ac`.

**Example:**

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
deployment-number = 12
client-policy = opengever.demo.fd
raven_tags = {"cluster": "demo.onegovgever.ch"}
```


## [ogds-postgres.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/ogds-postgres.cfg)

Uses PostgreSQL as database.
Be aware that the used database user is the same as the OS user.

**Variables:**
- `buildout:ogds-db-name`: name of the database

**Example:**

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/ogds-postgres.cfg
ogds-db-name = onegovgever-hinterfultigen
```


## [ogds-mysql.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/ogds-mysql.cfg)

Uses MySQL as database.

**Variables:**
- `buildout:ogds-db-name`: name of the database
- `buildout:ogds-db-user`: MySQL username
- `buildout:ogds-db-pw`: MySQL password

**Example:**

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/ogds-mysql.cfg
ogds-db-name = onegovgever-hinterfultigen
ogds-db-user = onegovgever-hinterfultigen
ogds-db-pw = PasswordF0rWhat?
```


## [tika-standalone.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/tika-standalone.cfg)

Use a standalone Tika server for this deployment.

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/tika-standalone.cfg
```


## [standard-sources.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/standard-sources.cfg)

The ``standard-sources.cfg`` is automatically included by the
``standard-deployment.cfg`` and makes sure that ``mr.developer`` is used and is
configured so that the standard deployment checkouts are applied.

Features:

- includes all public sources from [plonesource.org](http://plonesource.org)
- includes private sources [from our KGS](http://kgs.4teamwork.ch/sources.cfg)
- activates the `mr.developer` extension
- declares a ``buildout:deployment-checkouts`` variable, which contains all
  sources which have to be checked out on productive deployments; this list is
  automatically added to ``auto-checkout`` by default
- includes our PSC as source (find-links, authentication); this must be done
  here in order to avoid problems with buildout


## [source-master.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/source-master.cfg): Development- / Source-Deployments

Some deployments need to deploy `opengever.core` from source, which should include
using the source-version of all packages which are in the `auto-checkout` list of
`opengever.core`.
Such deployments must also use the `develop`-KGS.

For these deployments, the `source-master.cfg` can be included _instead of a
KGS_.

Example:

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-deployment.cfg
....
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/source-master.cfg
```

**WARNING:** The sources of the `master`-branch is extended because we cannot
  use the branch variable in the `extends`. You need to write your own
  `source-master.cfg` if you need to deploy another branch.


## [standard-dev.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/standard-dev.cfg)

The `standard-dev.cfg` is a standard development config to be used in
standardized GEVER policies. It may or may not be used by more sophisticated
policies.

The goal is to make the `development.cfg` in a standard GEVER policy as short as
possible

Usage example:
```ini
[buildout]
extends =
    test-policy.cfg
    https://raw.githubusercontent.com/4teamwork/ftw-buildouts/master/plone-development.cfg
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/standard-dev.cfg
    https://raw.githubusercontent.com/4teamwork/ftw-buildouts/master/bumblebee.cfg

ogds-db-name = opengeverftw
```


## [test-policy.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/test-policy.cfg)

The `test-policy.cfg` tests whether policy development works with the
`opengever.core` version installed in production.
The GEVER version installed in production might not be the newest, therefore the
test buildout in the policy package should extend a KGS version.

This config is usually extended in a GEVER policy in a config file with the same
name `test-policy.cfg`:

```ini
[buildout]
extends =
    https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/test-policy.cfg
    versions.cfg

package-name = opengever.ftw
```


## [test-core-development.cfg](https://github.com/4teamwork/gever-buildouts/blob/master/test-core-development.cfg)

The `test-core-development.cfg` tests whether the lastest `opengever.core`
development still works with a policy.

The goal is to notice when a change in `opengever.core` will break our policy so
that an upgrade will be problematic in the future.
This allows us to fix the problems when the knowledge is fresh, so that later
updates will be easy.


This config is usually extended in a GEVER policy in a config file with the same
name `test-core-development.cfg`:

```ini
[buildout]
extends = https://raw.githubusercontent.com/4teamwork/gever-buildouts/master/test-core-development.cfg
package-name = opengever.ftw
```
