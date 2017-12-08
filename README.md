# network_managment

This role configures your network interfaces automatically. This role also support complex setups with openvswitch and different vlans. On debian based systems a proper `/etc/network/interfaces` configuration is generated unless `network_managment_allways_script` is set to `True`.


## Requirements

A Linux distribution, with debian networking support or systemd as init system.
`python-netaddr` has to be installed on the executing machine.

## Role Variables

### Primary
| Option                                   | Type                         | Default | Description                                                                                                                                        | Required |
|------------------------------------------|------------------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------------------|:--------:|
| network_managment_allways_script         | boolean                      | `False` | Generate a systemd service and network management script always (disable networking.service if exist)                                              |     N    |
| network_managment_default_type           | [string](#type)              | `dhcp`  | Default type to setup a interface or bridge                                                                                                        |     N    |
| network_managment_default_dhcp_options   | [dict](#dhcp)                | `{}`    | Additional options for dhcp interfaces                                                                                                             |     N    |
| network_managment_default_static_options | [dict](#static)              | `{}`    | Defines how a static interface should be setup                                                                                                     |     N    |
| network_managment_gateway                | string                       |         | Add a default route via this gateway address                                                                                                       |     N    |
| network_managment_pre_up                 | string                       |         | Commands to execute before any other action is performed                                                                                           |     N    |
| network_managment_post_up                | string                       |         | Commands to execute after all other network operations are performed                                                                               |     N    |
| interfaces                               | [list of dicts](#interfaces) | `[]`    | List of all interfaces to setup, keep in mind it can cause various errors if you configure a interface here and later use it as a port on a bridge |     N    |
| bridges                                  | [list of dicts](#bridges)    | `[]`    | List of network bridges to setup (all bridges are managed by openvswitch)                                                                          |     N    |

### type
Defines how a network or bridge should be configured. Possible values are:

| value  | Description                            | Side effects                       |
|--------|----------------------------------------|------------------------------------|
| manual | Do nothing but bring up interface link | Ignore options `static` and `dhcp` |
| static | Configure interface statically         | Ignore option `dhcp`               |
| dhcp   | Configure interface via dhcp           | Ignore option `static`             |


### dhcp
| Option    | Type    | Default | Description                                                            | Required |
|-----------|---------|---------|------------------------------------------------------------------------|:--------:|
| hostname  | string  |         | Hostname to be requested                                               |     N    |
| leasetime | integer |         | Request a specific lease time in seconds.                              |     N    |
| metric    | integer |         | Metrics are used to prefer an interface over another one, lowest wins. |     N    |


### static
| Option  | Type   | Default | Description                     | Required |
|---------|--------|---------|---------------------------------|:--------:|
| ip      | string |         | IP Address to configure as CIDR |     Y    |
| gateway | string |         | Gateway                         |     N    |


### interfaces
| Option   | Type            | Default                                        | Description                                            | Required |
|----------|-----------------|------------------------------------------------|--------------------------------------------------------|:--------:|
| name     | string          |                                                | Name of the interface to configure                     |     Y    |
| bring_up | boolean         | `True`                                         | Automatically bring interface link up                  |     N    |
| type     | [string](#type) | `{{ network_managment_default_type }}`         | Specify how the network interface should be configured |     N    |
| dhcp     | [dict](#dhcp)   | `{{ network_managment_default_dhcp_options }}` | Additional options for dhcp interfaces                 |     N    |
| mtu      | integer         |                                                | MTU size                                               |     N    |


### bridges
Beside every option from the [interfaces](#interfaces) dict, the following options can be specified:

| Option | Type            | Default | Description                                      | Required |
|--------|-----------------|---------|--------------------------------------------------|:--------:|
| ports  | list of strings | `[]`    | List of physical interfaces to add to the bridge |     N    |


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) &gt;_markus.mroch@stuvus.uni-stuttgart.de_&lt;
