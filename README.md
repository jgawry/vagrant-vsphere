[![Build Status](https://travis-ci.org/nsidc/vagrant-vsphere.svg?branch=master)](https://travis-ci.org/nsidc/vagrant-vsphere)

# Vagrant vSphere Provider

This is a [Vagrant](http://www.vagrantup.com) 1.6.3+ plugin that adds a [vSphere](http://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.wssdk.apiref.doc_50%2Fright-pane.html)
provider to Vagrant, allowing Vagrant to control and provision machines using VMware. New machines are created from virtual machines or templates which must be configured prior to using using this provider.

This provider is built on top of the [RbVmomi](https://github.com/vmware/rbvmomi) Ruby interface to the vSphere API.

## Requirements
* Vagrant 1.6.3+
* VMware with vSphere API
* Ruby 1.9+
* libxml2, libxml2-dev, libxslt, libxslt-dev

## Current Version
**version: 0.18.0**

vagrant-vsphere (**version: 0.18.0**) is available from [RubyGems.org](https://rubygems.org/gems/vagrant-vsphere)

## Installation

Install using standard Vagrant plugin method:

```bash
vagrant plugin install vagrant-vsphere
```

This will install the plugin from RubyGems.org.

Alternatively, you can clone this repository and build the source with `gem build vSphere.gemspec`.
After the gem is built, run the plugin install command from the build directory.

### Potential Installation Problems

The requirements for [Nokogiri](http://nokogiri.org/) must be installed before the plugin can be installed. See the [Nokogiri tutorial](http://nokogiri.org/tutorials/installing_nokogiri.html) for
detailed instructions.

The plugin forces use of Nokogiri ~> 1.5 to prevent conflicts with older versions of system libraries, specifically zlib.

## Usage

After installing the plugin, you must create a vSphere box. The example_box directory contains a metadata.json file
that can be used to create a dummy box with the command:

```bash
tar cvzf dummy.box ./metadata.json
```

This can be installed using the standard Vagrant methods or specified in the Vagrantfile.

After creating the dummy box, make a Vagrantfile that looks like the following:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = 'dummy'
  config.vm.box_url = './example_box/dummy.box'

  config.vm.provider :vsphere do |vsphere|
    vsphere.host = 'HOST NAME OF YOUR VSPHERE INSTANCE'
    vsphere.compute_resource_name = 'YOUR COMPUTE RESOURCE'
    vsphere.resource_pool_name = 'YOUR RESOURCE POOL'
    vsphere.template_name = 'YOUR VM TEMPLATE'
    vsphere.name = 'NEW VM NAME'
    vsphere.user = 'YOUR VMWARE USER'
    vsphere.password = 'YOUR VMWARE PASSWORD'
  end
end
```

And then run `vagrant up --provider=vsphere`.

### Custom Box

The bulk of this configuration can be included as part of a custom box. See the [Vagrant documentation](http://docs.vagrantup.com/v2/boxes.html)
and the Vagrant [AWS provider](https://github.com/mitchellh/vagrant-aws/tree/master/example_box) for more information and an example.

### Supported Commands

Currently the only implemented actions are `up`, `halt`, `reload`, `destroy`,
and `ssh`.

`up` supports provisioning of the new VM with the standard Vagrant provisioners.


## Configuration

This provider has the following settings, all are required unless noted:

* `host` -  IP or name for the vSphere API
* `insecure` - _Optional_ verify SSL certificate from the host
* `user` - user name for connecting to vSphere
* `password` - password for connecting to vSphere. If no value is given, or the
  value is set to `:ask`, the user will be prompted to enter the password on
  each invocation.
* `data_center_name` - _Optional_ datacenter containing the computed resource, the template and where the new VM will be created, if not specified the first datacenter found will be used
* `compute_resource_name` - _Required if cloning from template_ the name of the host containing the resource pool for the new VM
* `resource_pool_name` - the resource pool for the new VM. If not supplied, and cloning from a template, uses the root resource pool
* `clone_from_vm` - _Optional_ use a virtual machine instead of a template as the source for the cloning operation
* `template_name` - the VM or VM template to clone
* `vm_base_path` - _Optional_ path to folder where new VM should be created, if not specified template's parent folder will be used
* `name` - _Optional_ name of the new VM, if missing the name will be auto generated
* `customization_spec_name` - _Optional_ customization spec for the new VM
* `data_store_name` - _Optional_ the datastore where the VM will be located
* `linked_clone` - _Optional_ link the cloned VM to the parent to share virtual disks
* `proxy_host` - _Optional_ proxy host name for connecting to vSphere via proxy
* `proxy_port` - _Optional_ proxy port number for connecting to vSphere via proxy
* `vlan` - _Optional_ vlan to connect the first NIC to
* `memory_mb` - _Optional_ Configure the amount of memory (in MB) for the new VM
* `cpu_count` - _Optional_ Configure the number of CPUs for the new VM
* `mac` - _Optional_ Used to set the mac address of the new VM

### Cloning from a VM rather than a template

To clone from an existing VM rather than a template, set `clone_from_vm` to true. If this value is set, `compute_resource_name` and `resource_pool_name` are not required.

### Template_Name

* The template name includes the actual template name and the directory path containing the template.
* **For example:** if the template is a directory called **vagrant-templates** and the template is called **ubuntu-lucid-template** the `template_name` setting would be:
```
vsphere.template_name = "vagrant-templates/ubuntu-lucid-template"
```
![Vagrant Vsphere Screenshot](https://raw.githubusercontent.com/nsidc/vagrant-vsphere/master/vsphere_screenshot.png)


### VM_Base_Path

* The new vagrant VM will be created in the same directory as the template it originated from.
* To create the VM in a directory other than the one where the template was located, include the **vm_base_path** setting.
* **For example:** if the machines will be stored in a directory called **vagrant-machines** the `vm_base_path` would be:
```
vsphere.vm_base_path = "vagrant-machines"
```

![Vagrant Vsphere Screenshot](https://raw.githubusercontent.com/nsidc/vagrant-vsphere/master/vsphere_screenshot.png)


### Setting a static IP address

To set a static IP, add a private network to your vagrant file:

```ruby
config.vm.network 'private_network', ip: '192.168.50.4'
```

The IP address will only be set if a customization spec name is given. The customization spec must have network adapter settings configured. For each private network specified, there needs to be a corresponding network adapter in the customization spec. An error  will be thrown if there are more networks than adapters.

### Auto name generation

The name for the new VM will be automagically generated from the Vagrant machine name, the current timestamp and a random number to allow for simultaneous executions.

This is useful if running Vagrant from multiple directories or if multiple machines are defined in the Vagrantfile.

### Setting the MAC address

To set a static MAC address, add a `vsphere.mac` to your `Vagrantfile`:

```ruby
vsphere.mac = '00:50:56:XX:YY:ZZ'
```

Take care to avoid using invalid or duplicate VMware MAC addresses, as this can
easily break networking.

## Example Usage

### FILE: Vagrantfile
```ruby
VAGRANT_INSTANCE_NAME   = "vagrant-vsphere"

Vagrant.configure("2") do |config|
  config.vm.box     = 'vsphere'
  config.vm.box_url = 'https://vagrantcloud.com/ssx/boxes/vsphere-dummy/versions/0.0.1/providers/vsphere.box'

  config.vm.hostname = VAGRANT_INSTANCE_NAME
  config.vm.define VAGRANT_INSTANCE_NAME do |d|
  end

  config.vm.provider :vsphere do |vsphere|
    vsphere.host                  = 'vsphere.local'
    vsphere.name                  = VAGRANT_INSTANCE_NAME
    vsphere.compute_resource_name = 'vagrant01.vsphere.local'
    vsphere.resource_pool_name    = 'vagrant'
    vsphere.template_name         = 'vagrant-templates/ubuntu14041'
    vsphere.vm_base_path          = "vagrant-machines"

    vsphere.user     = 'vagrant-user@vsphere'
    vsphere.password = '***************'
    vsphere.insecure = true
  end
end
```

### Vagrant Up
```bash
vagrant up --provider=vsphere
```

### Vagrant SSH
```bash
vagrant ssh
```

### Vagrant Destroy
```bash
vagrant destroy
```

## Version History
* 0.0.1
  * Initial release
* 0.1.0
  * Add folder syncing with guest OS
  * Add provisioning
* 0.2.0
  * Merge halt action from [catharsis](https://github.com/catharsis)
* 0.3.0
  * Lock Nokogiri version at 1.5.10 to prevent library conflicts
  * Add support for customization specs
* 0.4.0
  * Add support for specifying datastore location for new VMs
* 0.5.0
  * Allow setting static ip addresses using Vagrant private networks
  * Allow cloning from VM or template
* 0.5.1
  * fix rsync on Windows, adapted from [mitchellh/vagrant-aws#77](https://github.com/mitchellh/vagrant-aws/pull/77)
* 0.6.0
  * add support for the `vagrant ssh -c` command
* 0.7.0
  * handle multiple private key paths
  * add auto name generation based on machine name
  * add support for linked clones
* 0.7.1
  * fixes rsync error reporting
  * updates locales yaml
  * restricts rbvmomi dependency
* 0.7.2
  * includes template in get_location (from: tim95030 fixes issue #38)
  * updates Gemfile to fall back to old version of vagrant for if ruby < 2.0.0 is available.
* 0.8.0
  * Adds configuration for connecting via proxy server (tkak issue #40)
* 0.8.1
  * Fixes [#47](https://github.com/nsidc/vagrant-vsphere/issues/47) via [olegz-alertlogic #52](https://github.com/nsidc/vagrant-vsphere/pull/52)
* 0.8.2
  * fixes no error messages [#58 leth:no-error-message](https://github.com/nsidc/vagrant-vsphere/pull/58)
  * fixes typo [#57 targetx007](https://github.com/nsidc/vagrant-vsphere/pull/57)
  * fixes additional no error messages
* 0.8.3
  * Fixed "No error message" on rbvmomi method calls. [#74: mkuzmin:rbvmomi-error-messages](https://github.com/nsidc/vagrant-vsphere/pull/74)
* 0.8.4
  * Use root resource pool when cloning from template [#63: matt-richardson:support-resource-pools-on-vsphere-standard-edition](https://github.com/nsidc/vagrant-vsphere/pull/63)
* 0.8.5
  * fixed synced folders to work with WinRM communicator [#72 10thmagnitude:master](https://github.com/nsidc/vagrant-vsphere/pull/72)
* 0.9.0
  * increases Vagrant requirements to 1.6.3+
  * Supports differentiating between SSH/WinRM communicator [#67 marnovdm:feature/waiting-for-winrm](https://github.com/nsidc/vagrant-vsphere/pull/67)
* 0.9.1
  * reuse folder sync code from Vagrant core. [#66 mkuzmin:sync-folders](https://github.com/nsidc/vagrant-vsphere/pull/66)
* 0.9.2
  * Instruct vagrant to set the guest hostname according to Vagrantfile [#69 ddub:set-hostname](https://github.com/nsidc/vagrant-vsphere/pull/69)
* 0.10.0
  * new optional parameter to clone into custom folder in vSphere [#73 mikola-spb:vm-base-path](https://github.com/nsidc/vagrant-vsphere/pull/73)
  * follows [semver](http://semver.org/) better, this adds functionality in a
    backwards compatible way, so bumps the minor. 0.9.0, should have been a
    major version.
* 0.11.0
  * Create the VM target folder if it doesn't exist #76 marnovdm:feature/create_vm_folder.
* 0.12.0
  * Use a directory name where Vagrantfile is stored as a prefix for VM name [#82 mkuzmin:name-prefix](https://github.com/nsidc/vagrant-vsphere/pull/82).
* 0.13.0
  * Find and install box file for multi-provider boxes automatically [#86 mkuzmin:install-box](https://github.com/nsidc/vagrant-vsphere/pull/86) & [#87 mkuzmin/provider-name](https://github.com/nsidc/vagrant-vsphere/pull/87).
* 0.13.1
  * Change Nokogiri Major Version dependency [#90 highsineburgh:SAITRADLab-master](https://github.com/nsidc/vagrant-vsphere/pull/90)
* 0.14.0 Add vlan configuration [#91 rylarson:add-vlan-configuration](https://github.com/nsidc/vagrant-vsphere/pull/91)
  * Added a new configuration option 'vlan' that lets you specify the vlan string
  * If vlan is set, the clone spec is modified with an edit action to connect the first NIC on the VM to the configured VLAN.
* 0.15.0 Make destroy work in all vm states [#93 rylarson:make-destroy-work-in-all-vm-states](https://github.com/nsidc/vagrant-vsphere/pull/93) (fixes #77)
  * If the VM is powered on, then it is powered off, and destroyed.
  * If the VM is powered off, it is just destroyed.
  * If the VM is suspended, it is powered on, then powered off, then destroyed.
* 0.16.0 Add ability to configure amount of memory the new cloned VM will have [#94 rylarson:add-memory-configuration](https://github.com/nsidc/vagrant-vsphere/pull/94).
* 0.17.0
  * Add ability to configure the CPU Count
    [#96 rylarson:add-cpu-configuration](https://github.com/nsidc/vagrant-vsphere/pull/96).
  * Prompt the user to enter a password if none is given, or the configuration
    value is set to `:ask`
    [#97 topmedia:password-prompt](https://github.com/nsidc/vagrant-vsphere/pull/97).
  * Add support for `vagrant reload`
    [#105 clintoncwolfe:add-reload-action](https://github.com/nsidc/vagrant-vsphere/pull/105).
  * Fix compatibility with Vagrant 1.7 to use vSphere connection info from a base
    box
    [#111 mkuzmin:get-state](https://github.com/nsidc/vagrant-vsphere/pull/111).
* 0.18.0
  * Gracefully power off the VM with `vagrant halt`, and shutdown before
    deleting the VM with `vagrant destroy`
    [#104 clintoncwolfe:shutdown-guest-on-halt](https://github.com/nsidc/vagrant-vsphere/pull/104).
  * Add configuration option `mac` to specify a MAC address for the VM
    [#108 dataplayer:master](https://github.com/nsidc/vagrant-vsphere/pull/108).

## Versioning

This plugin follows the principles of [Semantic Versioning 2.0.0](http://semver.org/)

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

### Unit Tests

Please run the unit tests to verify your changes. To do this simply run `rake`.
If you want a quick merge, write a spec that fails before your changes are applied and that passes after.


If you don't have rake installed, first install [bundler](http://bundler.io/) and run `bundle install`.

### Development Without Building the Plugin

To test your changes when developing the plugin, you have two main
options. First, you can build and install the plugin from source every time you
make a change:

1. Make changes
2. `rake build`
3. `vagrant plugin install ./pkg/vagrant-vsphere-$VERSION.gem`
4. `vagrant up --provider=vsphere`

Second, you can use Bundler and the Vagrant gem to execute vagrant commands,
saving time as you never have to wait for the plugin to build and install:

1. Make changes
2. `bundle exec vagrant up --provider=vsphere`

This method uses the version of Vagrant specified in
[`Gemfile`](https://github.com/nsidc/vagrant-vsphere/blob/master/Gemfile). It
will also cause Bundler and Vagrant to output warnings every time you run
`bundle exec vagrant`, because `Gemfile` lists **vagrant-vsphere** twice (once
with `gemspec` and another time in the `group :plugins` block), and Vagrant
prefers to be run from the official installer rather than through the gem.

Despite those warning messages, this is the
[officially recommended](https://docs.vagrantup.com/v2/plugins/development-basics.html)
method for Vagrant plugin development.

## License

The Vagrant vSphere Provider is licensed under the MIT license. See [LICENSE.txt][license].

[license]: https://raw.github.com/nsidc/vagrant-vsphere/master/LICENSE.txt

## Credit

This software was developed by the National Snow and Ice Data Center with funding from multiple sources.
