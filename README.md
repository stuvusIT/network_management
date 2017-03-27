# firewall

Configures a Debian server as firewall and gateway server. Rules are either auto generated from all group and host vars, at your inventory or specified at 'template/main.ferm'.


## Requirements



## Role Variables
the following variables are only used from host and group vars, where this role does apply:
```yml
reverse_proxy: <primary host(ip) oder ansible hostname, where all https and http traffic should be forrwarded too>
```

the following variables are used from all host or group vars(not only from hosts where this role does apply):
```yml
#allow https(443) traffic internally between all servers(which take use of following option) and the reverse proxy.
served_domains:
  - domains:
    - domain: <domain name> #these values are currently ignored by this role(the role only checks if served_domains is defined or not)
    ...
  ...

#allow traffic for all specified protocol to all servers which take use of that option.
services:
  - <service name or port>
  ...
```


## Dependencies



## Example Playbook

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:


### Playbook

```yml
```


### Vars

```yml
```


### Result

A short summary what the playbook actually does.


## License

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).


## Author Information

 * [Markus Mroch (Mr. Pi)](https://github.com/Mr-Pi) _markus.mroch@stuvus.uni-stuttgart.de_
