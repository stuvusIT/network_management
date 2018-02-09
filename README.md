# network_management

This role configures your network interfaces automatically. This role also support complex setups with openvswitch and different vlans. On debian based systems a proper `/etc/network/interfaces` configuration is generated unless `network_management_always_script` is set to `True`.


## Requirements

A Linux distribution with debian networking support or systemd as init system.
`python-netaddr` has to be installed on the executing machine.

## Role Variables

### Primary
| Option                                    | Type                               | Default                  | Description                                                                                                                                           | Required |
|-------------------------------------------|------------------------------------|--------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|:--------:|
| `network_management_always_script`        | boolean                            | `False`                  | Always generate a systemd service and network management script (overwrites networking.service if it exists)                                          |     N    |
| `network_management_default_type`         | [string](#type)                    | `dhcp`                   | Default type to setup a interface or bridge                                                                                                           |     N    |
| `network_management_default_bring_up`     | boolean                            | `True`                   | Bring interface up on start                                                                                                                           |     N    |
| `network_management_default_dhcp_options` | [dict](#dhcp)                      | `{}`                     | Additional options for dhcp interfaces                                                                                                                |     N    |
| `network_management_default_gateway`      | string                             |                          | Add a default route via this gateway address                                                                                                          |     N    |
| `network_management_default_mtu`          | integer                            |                          | Detault MTU size to use for all interfaces                                                                                                            |     N    |
| `network_management_pre_up`               | string                             | `""`                     | Commands to execute before any other action is performed                                                                                              |     N    |
| `network_management_post_up`              | string                             | `""`                     | Commands to execute after all other network operations are performed                                                                                  |     N    |
| `network_management_nameservers`          | list of strings                    | `['8.8.8.8', '8.8.4.4']` | List of all nameservers to use                                                                                                                        |     N    |
| `network_management_clear_bridges`        | boolean                            | `False`                  | Remove all ovs bridges before recreating them. This is useful for renaming bridges.                                                                   |     N    |
| `network_management_reboot_for_config`    | boolean                            | `False`                  | Reboot target server to setup new network config, useful for major network configuration, which may require manual interactions otherwise             |     N    |
| `interfaces`                              | [list of dicts](#interfaces)       | `[]`                     | List of all interfaces to setup, keep in mind it can cause various errors if you configure a interface here and later use it as a port on a bridge    |     N    |
| `bridges`                                 | [list of dicts](#bridges)          | `[]`                     | List of network bridges to setup (all bridges are managed by openvswitch)                                                                             |     N    |
| `patch_field`                             | [list of key values](#patch_field) | `[]`                     | A list of network interfaces or bridge ports to patch together (ex. wire/patch one bridge port with one vlan to another bridge with a different vlan) |     N    |

### type
Defines how a network or bridge should be configured. Possible values are:

| Value    | Description                            | Side effects                                  |
|----------|----------------------------------------|-----------------------------------------------|
| `manual` | Do nothing but bring up interface link | Ignore options `ips`, `default_gw` and `dhcp` |
| `static` | Configure interface statically         | Ignore option `dhcp`                          |
| `dhcp`   | Configure interface via dhcp           | Ignore option `ips` and `default_gw`          |

### dhcp
| Option      | Type    | Default                  | Description                                                                        | Required |
|-------------|---------|--------------------------|------------------------------------------------------------------------------------|:--------:|
| `hostname`  | string  | `{{ ansible_hostname }}` | Hostname to be requested (is ignored in `network_management_default_dhcp_options`) |     N    |
| `leasetime` | integer |                          | Request a specific lease time in seconds.                                          |     N    |
| `metric`    | integer |                          | Metrics are used to prefer an interface over another one, lowest wins.             |     N    |

### interfaces
| Option        | Type            | Default                                         | Description                                                                                               |             Required            |
|---------------|-----------------|-------------------------------------------------|-----------------------------------------------------------------------------------------------------------|:-------------------------------:|
| `name`        | string          |                                                 | Name of the interface to configure, ignored if `mac` is specified                                         |  only if `mac` is not specified |
| `mac`         | string          |                                                 | MAC-Address of the interface to configure. If `name` is specified, `mac` is autodetected.                 | only if `name` is not specified |
| `bring_up`    | boolean         | `{{ network_management_default_bring_up }}`     | Automatically bring interface link up                                                                     |                N                |
| `type`        | [string](#type) | `{{ network_management_default_type }}`         | Specify how the network interface should be configured                                                    |                N                |
| `dhcp`        | [dict](#dhcp)   | `{{ network_management_default_dhcp_options }}` | Additional options for dhcp interfaces                                                                    |                N                |
| `nameservers` | list of strings | `{{ network_management_nameservers }}`          | Nameservers to use                                                                                        |                N                |
| `mtu`         | integer         |                                                 | MTU size                                                                                                  |                N                |
| `ip`          | string          |                                                 | IP address to assign as IP with CIDR suffix                                                               |                N                |
| `ips`         | list of strings |                                                 | IP addresses to assign (IP address with CIDR suffix)                                                      |    only if `type` is `static`   |
| `gateway`     | string          | `{{ network_management_default_gateway }}`      | Add a default route via this gateway address                                                              |                N                |
| `broadcast`   | string          | ___auto calculated___                           | Broadcast address to use                                                                                  |                N                |
| `hostname`    | string          | `{{ ansible_hostname }}`                        | Hostname to be requested (is ignored in `network_management_default_dhcp_options`). Only dhcp interfaces. |                N                |
| `leasetime`   | integer         |                                                 | Request a specific lease time in seconds. Only dhcp interfaces.                                           |                N                |
| `metric`      | integer         |                                                 | Metrics are used to prefer an interface over another one, lowest wins. Only dhcp interfaces.              |                N                |

### bridges
Beside every option from the [interfaces](#interfaces) dict, the following options can be specified:

| Option  | Type                   | Default | Description                                      | Required |
|---------|------------------------|---------|--------------------------------------------------|:--------:|
| `ports` | [list of dicts](#port) | `[]`    | List of physical interfaces to add to the bridge |     N    |

### port
| Option | Type    | Default | Description                                         | Required |
|--------|---------|---------|-----------------------------------------------------|:--------:|
| port   | string  |         | Interface name to add to bridge                     |     Y    |
| vlan   | integer |         | Add the port with specified vlan to selected bridge |     N    |

### patch_field
| Option      | Type   | Default | Description                                                   | Required |
|-------------|--------|---------|---------------------------------------------------------------|:--------:|
| ___key___   | string |         | First port or interface in between the link should be made    | Y        |
| ___value___ | string |         | Second port or interfaces in between the link should be made  | Y        |


## ToDo
* Implement script based network configuration
* Implement systemd networkd based network configuration


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) &lt;_markus.mroch@stuvus.uni-stuttgart.de_&gt;
