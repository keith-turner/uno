![Uno][logo]
---
[![Apache License][li]][ll]

[Apache Fluo][fluo] depends on [Apache Accumulo][accumulo], [Apache Zookeeper][zookeeper], and
[Apache Hadoop][hadoop]. Setting up these dependencies is time consuming. Uno provides a set of
helper scripts to automate setting up these dependencies on a single machine. This makes it quick
for a developer to experiment with Fluo in a realistic environment. 

Uno is designed for developers who need to frequently upgrade and test their code, and do not care
about preserving data. While Uno makes it easy to setup a dev stack running Fluo or Accumulo, it
also makes it easy clear your data and setup your dev stack again. To avoid inadvertent data loss,
Uno should not be used in production.

Checkout [Muchos] for setting up Fluo's dependencies on multiple machines.

## Requirements

Uno requires the following software to be installed on your machine.

* Java - JDK 8 is required for running Fluo.
* wget - Needed for `fetch` command to download tarballs.
* Maven - Only needed if `fetch` command builds tarball from local repo.

You should also be able to [ssh to localhost without a passphrase][ssh-docs].

## Installation

First, clone the Uno repo on a local disk with enough space to run Hadoop, Accumulo, etc:

    git clone https://github.com/astralway/uno.git

The `uno` command uses `conf/env.sh.example` for its default configuration which should be
sufficient for most users.

Optionally, you can customize this configuration by creating an `env.sh` file and modifying it for
your environment:

```bash
cd conf/
cp env.sh.example env.sh
vim env.sh
```

Uno can optionally setup a metrics/monitoring tool (i.e Grafana+InfluxDB) that can be used to
monitor your Apache Fluo applications. This setup does not occur with the default configuration. You
must set `SETUP_METRICS` to `true` in your `env.sh`.

All commands are run using the `uno` script in `bin/`. Uno has a command that helps you configure
your shell so that you can run commands from any directory and easily set common environment
variables in your shell for Uno, Hadoop, Zookeeper, Fluo, and Spark. Run the following command to
print this shell configuration. You can also add `--paths` or `--vars` to the command below to limit
output to PATH or environment variable configuration:

    uno env

You can either copy and paste this output into your shell or add the following (with a correct path)
to your ~/.bashrc automatically configure every new shell.

```bash
eval "$(/path/to/uno/bin/uno env)"
```

With `uno` script set up, you can now use it to download, configure, and run Fluo's dependencies.

## Fetch command

The `uno fetch` command fetches the binary tarball dependencies of Apache Fluo to be later installed
by the `setup` command. By default, it will download binary tarballs. However, you can configure the
`fetch` command to build Fluo or Accumulo from a local git repo by setting `FLUO_REPO` or
`ACCUMULO_REPO` in `env.sh`.

If `uno fetch all` is run, all dependencies will be either downloaded or built. If you would like to
only fetch certain dependencies, run `uno fetch` to see a list of possible dependencies.

After the `fetch` command is run for the first time, it only needs to run again if you want to
upgrade dependencies and need to download/build the latest version.

## Setup command

The `setup` command will install the downloaded tarballs to the directory set by `$INSTALL` in your
env.sh and run you local development cluster. The command can be run in several different ways:

1. Sets up Apache Accumulo and its dependencies of Hadoop, Zookeeper. This starts all processes and
   will wipe Accumulo/Hadoop if this command was run previously. This command also sets up Spark
   and starts Spark's History Server (set `START_SPARK_HIST_SERVER=false` in your env.sh to turn 
   off). This command is useful if you are using Uno for Accumulo development.

        uno setup accumulo

2. Sets up Apache Fluo along with Accumulo (and its dependencies). It also sets up a metrics server
   for Fluo consisting of InfluxDB & Grafana if `SETUP_METRICS` is set to true in env.sh. This
   command will wipe your cluster. While Fluo is set up, it does not start any Fluo applications.

        uno setup fluo

3. Sets up Apache Fluo only. This will stop any previously running Fluo applications but it will not
   wipe your cluster. If you want upgrade Fluo without wiping your cluster, run `uno fetch fluo`
   before running this command.

        uno setup fluo-only

4. Sets up metrics service (InfluxDB + Grafana). This command will set up metrics even if
   `SETUP_METRICS` is set to false in env.sh.

        uno setup metrics

You can confirm that everything started by checking the monitoring pages below:

 * [Hadoop NameNode](http://localhost:50070/)
 * [Hadoop ResourceManager](http://localhost:8088/)
 * [Accumulo Monitor](http://localhost:50095/)
 * [Spark HistoryServer](http://localhost:18080/)
 * [Grafana](http://localhost:3000/) (optional)
 * [InfluxDB Admin](http://localhost:8083/) (optional)

You can verify that Fluo was installed by correctly by running the `fluo` command which you can use
to administer Fluo:

    ./install/fluo-1.0.0-beta-1/bin/fluo

If you run some tests and then want a fresh cluster, run the `setup` command again which will
kill all running processes, clear any data and logs, and restart your cluster.

## Running Apache Fluo applications

Before running an Apache Fluo application, it is recommended that you configure your shell using
`uno env`. If this is done, many Fluo example applications (such as [Webindex] and [Phrasecount])
can be run by simply cloning their repo and executing their start scripts (which will use
environment variables set in your shell by `uno env`).

If you want to create your own Fluo application, you should mimic the scripts of example Fluo
applications or follow the instructions starting at the [Configure a Fluo application][configure]
section of the Fluo install instructions. These instructions will guide you through the process of
configuring, initializing, and starting your application.

[fluo]: http://fluo.apache.org/
[accumulo]: http://accumulo.apache.org/
[zookeeper]: http://zookeeper.apache.org/
[hadoop]: http://hadoop.apache.org/
[mirrors]: http://www.apache.org/dyn/closer.cgi
[Webindex]: https://github.com/astralway/webindex
[Phrasecount]: https://github.com/astralway/phrasecount
[configure]: https://github.com/apache/fluo/blob/master/docs/install.md#configure-a-fluo-application
[li]: http://img.shields.io/badge/license-ASL-blue.svg
[ll]: https://github.com/astralway/uno/blob/master/LICENSE
[logo]: contrib/uno-logo.png
[Muchos]: https://github.com/astralway/muchos
[ssh-docs]: https://hadoop.apache.org/docs/r2.7.2/hadoop-project-dist/hadoop-common/SingleCluster.html#Setup_passphraseless_ssh
