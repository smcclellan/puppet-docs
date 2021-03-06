---
layout: default
title: "PE 1.2 Manual: The accounts Class"
canonical: "/pe/latest/accounts_class.html"
---

{% include pe_1.2_nav.markdown %}

[knownissues]: ./known_issues.html#accounts-class-requires-an-inert-variablefile

The `accounts` Class
=====

The `accounts` class can do any or all of the following:

- Create and manage a set of `accounts::user` resources
- Create and manage a set of shared `group` resources
- Maintain a pair of rules in the `sudoers` file granting privileges to the `sudo` and `sudonopw` groups

This class is designed for cases where your account data is **maintained separately** from your Puppet manifests. This usually means it is being **read on demand from a non-Puppet directory service or CMDB, managed manually by a user who does not write Puppet code,** or **generated by an out-of-band process.** If your site's account data will be maintained manually by a sysadmin able to write Puppet code, it will make more sense to maintain it as a normal set of `accounts::user` and `group` resources, although you may still wish to use the `accounts` class to maintain `sudoers` rules. 

To manage users and groups with the `accounts` class, you **must** prepare a data store and configure the class for the data store when you declare it. 

Usage Example
-----

To use YAML files as a data store: 

{% highlight ruby %}
    class {'accounts':
      data_store => yaml,
    }
{% endhighlight %}

To use a Puppet class as a data store and manage `sudoers` rules:

{% highlight ruby %}
    class {'accounts':
      data_store     => namespace,
      data_namespace => 'site::accounts::data',
      manage_sudoers => true,
    }
{% endhighlight %}

To manage `sudoers` rules without managing any users or groups:

{% highlight ruby %}
    class {'accounts':
      manage_users   => false,
      manage_groups  => false,
      manage_sudoers => true,
    }
{% endhighlight %}

Data Stores
-----

Account data can come from one of two sources: **a Puppet class that declares three variables,** or **a set of three [YAML](http://yaml.org/) files** stored in `/etc/puppetlabs/puppet/data`.

### Using a Puppet Class as a Data Store

This option is most useful if you are able to generate or import your user data with a [custom function](/guides/custom_functions.html), which may be querying from an LDAP directory or some other arbitrary data source.

The Puppet class containing the data **must have a name ending in `::data`.** (We recommend `site::accounts::data`.) This class must declare the following variables: 

- `$users_hash` should be a hash in which each key is the **title of an `accounts::user` resource** and each value is **a hash containing that resource's attributes and values.**
- `$groups_hash` should be a hash in which each key is the **title of a group** and each value is **a hash containing that resource's attributes and values.**
- `$users_hash_default` is an inert variable resulting from a [known issue][knownissues], and should be an empty hash (`{}`). **Note that part of this variable's name is the reverse of the corresponding file name.**

[See below][dataformat] for examples of the data formats used in these variables.

When declaring the `accounts` class to use data in a Puppet class, use the following attributes: 

    data_store     => namespace,
    data_namespace => {name of class},

### Using YAML Files as a Data Store

This option is most useful if your user data is being generated by an out-of-band process or is being maintained by a user who does not write Puppet manifests.

When storing data in YAML, the following valid [YAML](http://yaml.org/) files must exist in `/etc/puppetlabs/puppet/data`:

- `accounts_users_hash.yaml`, which should contain an anonymous hash in which each key is the **title of an `accounts::user` resource** and each value is **a hash containing that resource's attributes and values.**
- `accounts_groups_hash.yaml`, which should contain an anonymous hash in which each key is the **title of a group** and each value is **a hash containing that resource's attributes and values.**
- `accounts_users_default_hash.yaml`, which is an inert file resulting from a [known issue][knownissues] and should contain an anonymous hash with at least one dummy key/value pair. **Note that part of this file's name is the reverse of the corresponding Puppet variable name.**

[See below][dataformat] for examples of the data formats used in these variables.

When declaring the `accounts` class to use data in YAML files, use the following attribute: 

    data_store => yaml,

### Data Formats

[dataformat]: #data-formats

This class uses three hashes of data to construct the `accounts::user` and `group` resources it manages. 

#### The Users Hash

The **users hash** represents a set of `accounts::user` resources. Each key should be the **title of an `accounts::user` resource,** and each value should be **another hash containing that resource's attributes and values.** 

##### Puppet Example

{% highlight ruby %}
    $users_hash = {
      sysop => {
        locked  => false,
        comment => 'System Operator',
        uid     => '700',
        gid     => '700',
        groups  => ['admin', 'sudonopw'],
        sshkeys => ['ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAwLBhQefRiXHSbVNZYKu2o8VWJjZJ/B4LqICXuxhiiNSCmL8j+5zE/VLPIMeDqNQt8LjKJVOQGZtNutW4OhsLKxdgjzlYnfTsQHp8+JMAOFE3BD1spVnGdmJ33JdMsQ/fjrVMacaHyHK0jW4pHDeUU3kRgaGHtX4TnC0A175BNTH9yJliDvddRzdKR4WtokNzqJU3VPtHaGmJfXEYSfun/wFfc46+hP6u0WcSS7jZ2WElBZ7gNO4u2Z+eJjFWS9rjQ/gNE8HHlvmN0IUuvdpKdBlJjzSiKZR+r/Bo9ujQmGY4cmvlvgmcdajM/X1TqP6p3OuouAk5QSPUlDRV91oEHw== sysop+moduledevkey@puppetlabs.com'],
      },
      villain => {
        locked  => true,
        comment => 'Test Locked Account',
        uid     => '701',
        gid     => '701',
        groups  => ['admin', 'sudonopw'],
        sshkeys => ['ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAwLBhQefRiXHSbVNZYKu2o8VWJjZJ/B4LqICXuxhiiNSCmL8j+5zE/VLPIMeDqNQt8LjKJVOQGZtNutW4OhsLKxdgjzlYnfTsQHp8+JMAOFE3BD1spVnGdmJ33JdMsQ/fjrVMacaHyHK0jW4pHDeUU3kRgaGHtX4TnC0A175BNTH9yJliDvddRzdKR4WtokNzqJU3VPtHaGmJfXEYSfun/wFfc46+hP6u0WcSS7jZ2WElBZ7gNO4u2Z+eJjFWS9rjQ/gNE8HHlvmN0IUuvdpKdBlJjzSiKZR+r/Bo9ujQmGY4cmvlvgmcdajM/X1TqP6p3OuouAk5QSPUlDRV91oEHw== villain+moduledevkey@puppetlabs.com'],
      },
    }
{% endhighlight %}

##### YAML Example

    --- 
    sysop:
      locked: false
      comment: System Operator
      uid: '700'
      gid: '700'
      groups:
      - admin
      - sudonopw
      sshkeys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAwLBhQefRiXHSbVNZYKu2o8VWJjZJ/B4LqICXuxhiiNSCmL8j+5zE/VLPIMeDqNQt8LjKJVOQGZtNutW4OhsLKxdgjzlYnfTsQHp8+JMAOFE3BD1spVnGdmJ33JdMsQ/fjrVMacaHyHK0jW4pHDeUU3kRgaGHtX4TnC0A175BNTH9yJliDvddRzdKR4WtokNzqJU3VPtHaGmJfXEYSfun/wFfc46+hP6u0WcSS7jZ2WElBZ7gNO4u2Z+eJjFWS9rjQ/gNE8HHlvmN0IUuvdpKdBlJjzSiKZR+r/Bo9ujQmGY4cmvlvgmcdajM/X1TqP6p3OuouAk5QSPUlDRV91oEHw== sysop+moduledevkey@puppetlabs.com
    villain:
      locked: true
      comment: Test Locked Account
      uid: '701'
      gid: '701'
      groups:
      - admin
      - sudonopw
      sshkeys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAwLBhQefRiXHSbVNZYKu2o8VWJjZJ/B4LqICXuxhiiNSCmL8j+5zE/VLPIMeDqNQt8LjKJVOQGZtNutW4OhsLKxdgjzlYnfTsQHp8+JMAOFE3BD1spVnGdmJ33JdMsQ/fjrVMacaHyHK0jW4pHDeUU3kRgaGHtX4TnC0A175BNTH9yJliDvddRzdKR4WtokNzqJU3VPtHaGmJfXEYSfun/wFfc46+hP6u0WcSS7jZ2WElBZ7gNO4u2Z+eJjFWS9rjQ/gNE8HHlvmN0IUuvdpKdBlJjzSiKZR+r/Bo9ujQmGY4cmvlvgmcdajM/X1TqP6p3OuouAk5QSPUlDRV91oEHw== villain+moduledevkey@puppetlabs.com

#### The Groups Hash

The **groups hash** represents a set of shared `group` resources. Each key should be the **title of a `group` resource,** and each value should be **another hash containing that resource's attributes and values.**

##### Puppet Example

{% highlight ruby %}
    $groups_hash = {
      developer => {
        gid    => 3003,
        ensure => present,
      },
      sudonopw => {
        gid    => 3002,
        ensure => present,
      },
      sudo     => {
        gid    => 3001,
        ensure => present,
      },
      admin    => {
        gid    => 3000,
        ensure => present,
      },
    }
{% endhighlight %}

##### YAML Example

    ---
    developer:
     gid: "3003"
     ensure: "present"
    sudonopw:
     gid: "3002"
     ensure: "present"
    sudo:
     gid: "3001"
     ensure: "present"
    admin:
     gid: "3000"
     ensure: "present"

#### The User Defaults Hash

The **user defaults hash** has no effect on the behavior of the `accounts` module, but is required due to a [known issue][knownissues]. It should be either an empty hash or a hash with one or more dummy key/value pairs. (The function that loads YAML from disk is unable to recognize an empty hash, as an empty YAML hash is indistinguishable from an empty YAML string.)

##### Puppet Example

{% highlight ruby %}
    $users_hash_default = {}
{% endhighlight %}

##### YAML Example

    ---
    key: "value"

Parameters
-----

### `manage_groups`

Whether to manage a set of shared groups, which can be used by all `accounts::user` resources. If true, your data store must define these groups in the `$groups_hash` variable or the `accounts_groups_hash.yaml` file. Allowed values are `true` and `false`; defaults to `true`.

### `manage_users`

Whether to manage a set of `accounts::user` resources. If true, your data store must define these users in the `$users_hash` variable or the `accounts_users_hash.yaml` file. Default attributes that apply to all users should be defined in the `$users_hash_default` variable or the `accounts_users_default_hash.yaml` file. Allowed values are `true` and `false`; defaults to `true`.

### `manage_sudoers`

Whether to add sudo rules to the node's `sudoers` file. If true, the class will add <!-- TK why the % sign --> `%sudo` and `%sudonopw` groups to the `sudoers` file and give them full sudo and passwordless sudo privileges respectively. You will need to make sure that the `sudo` and `sudonopw` groups exist in the groups hash, and that your chosen users have those groups in their `groups` arrays. Managing `sudoers` is not supported on Solaris.

Allowed values are `true` and `false`; defaults to `false`.

### `data_store`

Which data store to use for accounts and groups.

When set to `namespace`, data will be read from the puppet class specified in the `data_namespace` parameter.  When set to `yaml`, data will be read from specially-named YAML files in the `/etc/puppetlabs/puppet/data` directory. (If you have changed your `$confdir`, it will look in `$confdir/data`.) Example YAML files are provided in the `ext/data/` directory of this module.

Allowed values are `yaml` and `namespace`; defaults to `namespace`.

### `data_namespace`

The Puppet namespace from which to read data. This must be the name of a Puppet class, and must end with `::data` (we recommend using `site::accounts::data`), which will automatically be declared by the `accounts` class. The class must be a non-parameterized class that declares variables named:

- `$users_hash`
- `$groups_hash`
- `$users_hash_default`

See the `accounts::data` class included in this module (in `manifests/data.pp`) for an example; see below for information on each hash's data structure. 

Defaults to `accounts::data`.

### `sudoers_path`

The path to the `sudoers` file on this system. Defaults to `/etc/sudoers`.

