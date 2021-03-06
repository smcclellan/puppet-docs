---
layout: default
title: "Config Files: environment.conf"
canonical: "/puppet/latest/reference/config_file_environment.html"
---

[directory environments]: ./environments.html
[environmentpath]: ./environments.html#about-environmentpath
[modulepath]: /references/3.6.latest/configuration.html#modulepath
[manifest]: /references/3.6.latest/configuration.html#manifest
[config_version]: /references/3.6.latest/configuration.html#configversion
[environment_timeout]: /references/3.6.latest/configuration.html#environmenttimeout

When using [directory environments][], each environment may contain an `environment.conf` file. This file can override several settings whenever the puppet master is serving nodes assigned to that environment.

## Location

Each environment.conf file should be stored in a [directory environment][directory environments]. It should be at the top level of its home environment, next to the `manifests` and `modules` directories.

For example: if your [`environmentpath` setting][environmentpath] is set to `$confdir/environments`, the environment.conf file for the `test` environment should be located at `$confdir/environments/test/environment.conf`.

## Example

    # /etc/puppetlabs/puppet/environments/test/environment.conf

    # Puppet Enterprise requires $basemodulepath; see note below under "modulepath".
    modulepath = site:dist:modules:$basemodulepath

    # Use our custom script to get a git commit for the current state of the code:
    config_version = get_environment_commit.sh

## Format

The environment.conf file uses the same INI-like format as puppet.conf, with one exception: it cannot contain config sections like `[main]`. All settings in environment.conf must be outside any config section.

### Allowed Settings

In this version of Puppet, the environment.conf file is only allowed to override four settings:

* [`modulepath`][modulepath] --- **Note:** if you're using Puppet Enterprise, you must always include either `$basemodulepath` or `/opt/puppet/share/puppet/modules` in the modulepath, since PE uses the modules in `/opt` to configure orchestration and other features.
* [`manifest`][manifest]
* [`config_version`][config_version]
* [`environment_timeout`][environment_timeout]

### Relative Paths in Values

Most of the allowed settings accept **file paths** or **lists of paths** as their values.

If any of these paths are **relative paths** --- that is, they start _without_ a leading slash or drive letter --- they will be resolved relative to that environment's main directory.

For example:

* Environment directory: `/etc/puppetlabs/puppet/environments/test`
* Relative setting in environment.conf: `config_version = get_environment_commit.sh`
* Equivalent value for setting: `config_version = /etc/puppetlabs/puppet/environments/test/get_environment_commit.sh`

### Interpolation in Values

The settings in environment.conf can the values of other settings as variables (e.g. `$confdir`). Additionally, they can use the special `$environment` variable, which gets replaced with the name of the active environment.

The most useful variables to interpolate into environment.conf settings are:

* `$basemodulepath` --- useful for including the default module directories in the `modulepath` setting. Puppet Enterprise users should usually include this in the value of `modulepath`, since PE uses modules in the `basemodulepath` to configure orchestration and other features.
* `$environment` --- useful for locating files, or as a command line argument to your `config_version` script.
* `$confdir` --- useful for locating files.
