# network_management

This role configures your network interfaces automatically. This role also support complex setups with openvswitch and different vlans. On debian based systems a proper `/etc/network/interfaces` configuration is generated unless `network_management_always_script` is set to `True`.


## Requirements

A Linux distribution with debian networking support or systemd as init system.
`python-netaddr` has to be installed on the executing machine.

## Role Variables

### Primary
| Option                                     | Type                               | Default                  | Description                                                                                                                                                         | Required |
|:-------------------------------------------|:-----------------------------------|:-------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|:--------:|
| `network_management_default_type`          | [string](#type)                    | `dhcp`                   | Default type to setup a interface or bridge                                                                                                                         |     N    |
| `network_management_default_bring_up`      | boolean                            | `True`                   | Bring interface up on start                                                                                                                                         |     N    |
| `network_management_default_dhcp_options`  | [dict](#dhcp)                      | `{}`                     | Additional options for dhcp interfaces                                                                                                                              |     N    |
| `network_management_default_gateway`       | string                             |                          | Add a default route via this gateway address                                                                                                                        |     N    |
| `network_management_default_mtu`           | integer                            |                          | Detault MTU size to use for all interfaces                                                                                                                          |     N    |
| `network_management_default_src_address`   | string                             |                          | Source Address to use for outgoing addresses, only used if `network_management_default_gateway` is specified, the source address has to be assigned to an interface |     N    |
| `network_management_default_allow_hotplug` | boolean                            | `False`                  | Allow hotplugging (`True`) or configure automatically (`False`)                                                                                                     |     N    |
| `network_management_pre_up`                | string                             |                          | Shell Commands to execute before any other action is performed                                                                                                      |     N    |
| `network_management_post_up`               | string                             |                          | Shell Commands to execute after all other network operations are performed                                                                                          |     N    |
| `network_management_pre_down`              | string                             |                          | Shell Commands to execute before network goes down                                                                                                                  |     N    |
| `network_management_post_down`             | string                             |                          | Shell Commands to execute after network has gone down                                                                                                               |     N    |
| `network_management_nameservers`           | list of strings                    | `['8.8.8.8', '8.8.4.4']` | List of all nameservers to use                                                                                                                                      |     N    |
| `network_management_clear_bridges`         | boolean                            | `False`                  | Remove all ovs bridges before recreating them. This is useful for renaming bridges.                                                                                 |     N    |
| `network_management_reboot_for_config`     | boolean                            | `False`                  | Reboot target server to setup new network config, useful for major network configuration, which may require manual interactions otherwise                           |     N    |
| `network_management_default_cidr`          | string                             |                          | Default CIDR suffix to use, with leading '/' (ex. '/24')                                                                                                            |     N    |
| `network_management_default_port_type`     | string                             |                          | Default type for interfaces/ports. For valid values see `man ovs-vswitchd.conf.db`                                                                                  |     N    |
| `network_management_default_port_options`  | list of strings                    |                          | Default ovs options to set for new interfaces/ports. For valid options see `man ovs-vswitchd.conf.db`                                                               |     N    |
| `network_management_plain_run`             | boolean                            | `False`                  | Only write network configuration, do not apply them                                                                                                                 |     N    |
| `interfaces`                               | [list of dicts](#interfaces)       | `[]`                     | List of all interfaces to setup, keep in mind it can cause various errors if you configure a interface here and later use it as a port on a bridge                  |     N    |
| `bridges`                                  | [list of dicts](#bridges)          | `[]`                     | List of network bridges to setup (all bridges are managed by openvswitch)                                                                                           |     N    |
| `patch_field`                              | [list of key values](#patch_field) | `[]`                     | A list of network interfaces or bridge ports to patch together (ex. wire/patch one bridge port with one vlan to another bridge with a different vlan)               |     N    |
| `network_management_routes`                | [dict](#network_management_routes) |                          | A list of additional routes to set                                                                                                                                  |     N    |
| `network_management_tables`                | [dict](#network_management_tables) | `{}`                     | A list of all custom routing tables                                                                                                                                 |     N    |
| `network_management_disable_ipv6`          | boolean                            | `False`                  | Disables IPv6                                                                                                                                                       |     N    |
| `network_management_ipv4_forwarding`       | boolean                            | `False`                  | Enables IPv4 forwarding                                                                                                                                             |     N    |

### type
Defines how a network or bridge should be configured. Possible values are:

| Value    | Description                            | Side effects                                  |
|:---------|:---------------------------------------|:----------------------------------------------|
| `manual` | Do nothing but bring up interface link | Ignore options `ips`, `default_gw` and `dhcp` |
| `static` | Configure interface statically         | Ignore option `dhcp`                          |
| `dhcp`   | Configure interface via dhcp           | Ignore option `ips` and `default_gw`          |

### dhcp
| Option      | Type    | Default                  | Description                                                                        | Required |
|:------------|:--------|:-------------------------|:-----------------------------------------------------------------------------------|:--------:|
| `hostname`  | string  | `{{ ansible_hostname }}` | Hostname to be requested (is ignored in `network_management_default_dhcp_options`) |    N     |
| `leasetime` | integer |                          | Request a specific lease time in seconds.                                          |    N     |
| `metric`    | integer |                          | Metrics are used to prefer an interface over another one, lowest wins.             |    N     |

### interfaces
| Option          | Type            | Default                                          | Description                                                                                                          |            Required             |
|:----------------|:----------------|:-------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------|:-------------------------------:|
| `name`          | string          |                                                  | Name of the interface to configure, ignored if `mac` is specified                                                    | only if `mac` is not specified  |
| `mac`           | string          |                                                  | MAC-Address of the interface to configure. If `name` is specified, `mac` is autodetected.                            | only if `name` is not specified |
| `bring_up`      | boolean         | `{{ network_management_default_bring_up }}`      | Automatically bring interface link up                                                                                |                N                |
| `type`          | [string](#type) | `{{ network_management_default_type }}`          | Specify how the network interface should be configured                                                               |                N                |
| `nameservers`   | list of strings | `{{ network_management_nameservers }}`           | Nameservers to use                                                                                                   |                N                |
| `mtu`           | integer         |                                                  | MTU size                                                                                                             |                N                |
| `ip`            | string          |                                                  | IP address to assign (CIDR suffix is required if `network_management_default_cird` is not specified)                 |                N                |
| `ips`           | list of strings |                                                  | IP addresses to assign (CIDR suffix is required if `network_management_default_cidr` is not specified)               |   only if `type` is `static`    |
| `gateway`       | string          |                                                  | Gateway address to use for this interface                                                                            |                N                |
| `broadcast`     | string          | ___auto calculated___                            | Broadcast address to use                                                                                             |                N                |
| `hostname`      | string          | `{{ ansible_hostname }}`                         | Hostname to be requested (`network_management_default_dhcp_options` don't apply here). Only used on dhcp interfaces. |                N                |
| `leasetime`     | integer         |                                                  | Request a specific lease time in seconds. Only used on dhcp interfaces.                                              |                N                |
| `metric`        | integer         |                                                  | Metrics are used to prefer an interface over another one, lowest wins. Only used on dhcp interfaces.                 |                N                |
| `allow_hotplug` | boolean         | `{{ network_management_default_allow_hotplug }}` | Allow hotplugging (`True`) or configure automatically (`False`)                                                      |                N                |

### bridges
Beside every option from the [interfaces](#interfaces) dict, the following options can be specified:

| Option          | Type                   | Default                                          | Description                                                     | Required |
|:----------------|:-----------------------|:-------------------------------------------------|:----------------------------------------------------------------|:--------:|
| `ports`         | [list of dicts](#port) | `[]`                                             | List of physical interfaces to add to the bridge                |    N     |
| `allow_hotplug` | boolean                | `{{ network_management_default_allow_hotplug }}` | Allow hotplugging (`True`) or configure automatically (`False`) |    N     |

### port
| Option          | Type            | Default                                          | Description                                                     | Required |
|:----------------|:----------------|:-------------------------------------------------|:----------------------------------------------------------------|:--------:|
| `port`          | string          |                                                  | Interface name to add to bridge                                 |    Y     |
| `vlan`          | integer         |                                                  | Add the port with specified vlan to selected bridge             |    N     |
| `type`          | string          | `{{ network_management_default_port_type }}`     | Type of the port to add                                         |    N     |
| `options`       | list of strings | `{{ network_management_default_port_options }}`  | Additional ovs options for new port, type must been seeded      |    N     |
| `allow_hotplug` | boolean         | `{{ network_management_default_allow_hotplug }}` | Allow hotplugging (`True`) or configure automatically (`False`) |    N     |

### patch_field
| Option      | Type   | Default | Description                                                  | Required |
|:------------|:-------|:--------|:-------------------------------------------------------------|:--------:|
| ___key___   | string |         | First port or interface in between the link should be made   |    Y     |
| ___value___ | string |         | Second port or interfaces in between the link should be made |    Y     |

### network_management_routes
| Option      | Type                   | Default | Description                                     | Required |
|:------------|:-----------------------|:--------|:------------------------------------------------|:--------:|
| ___key___   | string                 |         | IP address or IP network as CIDR                |    Y     |
| ___value___ | [dict](#route_options) |         | Route option to set for specified network or IP |    Y     |

#### route_options
| Option    | Type    | Default                               | Description                                                                                               | Required |
|:----------|:--------|:--------------------------------------|:----------------------------------------------------------------------------------------------------------|:--------:|
| gateway   | string  |                                       | IP address to use as gateway                                                                              |     N    |
| interface | string  |                                       | Interface/device to us to forward traffic                                                                 |     N    |
| source    | string  |                                       | Source IP address to us to forward traffic                                                                |     N    |
| metric    | integer |                                       | Metric to use for this route definition                                                                   |     N    |
| mtu       | integer | `{{ network_management_default_mtu}}` | MTU to use for this route, this is done by iptroute2 (ex. `ip route add 1.2.3.4/32 via 1.1.1.1 mtu 1500`) |     N    |

### network_management_tables
| Option      | Type                           | Default | Description                      | Required |
|:------------|:-------------------------------|:--------|:---------------------------------|:--------:|
| ___key___   | string                         |         | Name of the custom routing table |     Y    |
| ___value___ | [dict](#routing_table_options) |         | Routing table options            |     Y    |


#### routing_table_options
For additional information see [iproute2 documentation](http://www.policyrouting.org/iproute2.doc.html#ss9.6).

| Option | Type                                  | Default | Description                                            | Required |
|:-------|:--------------------------------------|:--------|:-------------------------------------------------------|:--------:|
| id     | integer                               |         | routing table identifier                               |     Y    |
| rules  | [list of dicts](#routing_table_rules) |         | rules to identify traffic to be routed with this table |     N    |

#### routing_table_rules
| Option    | Type            | Default | Description                                                                          | Required |
|:----------|:----------------|:--------|:-------------------------------------------------------------------------------------|:--------:|
| type      | string          |         | type of this rule (one of `unicast`, `blackhole`, `unreachable`, `prohibit`, `nat`)  |     N    |
| from      | string          |         | select source prefix to match                                                        |     N    |
| to        | string          |         | select destination prefix to match                                                   |     N    |
| interface | string          |         | incoming interface to match (`lo` for local traffic)                                 |     N    |
| nat       | string          |         | the base of IP address block to translate source address                             |     N    |
| tos       | integer/string  |         | select TOS value to match                                                            |     N    |
| fwmark    | integer/string  |         | select value of fwmark to match                                                      |     N    |
| priority  | integer (32bit) |         | priority of this rule. Each rule should have an explicitly set unique priority value |     N    |
| realms    | string          |         | realms to select if the rule matched and routing table lookup succeeded              |     N    |


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) &lt;_markus.mroch@stuvus.uni-stuttgart.de_&gt;
