This is my personal archive for libvirt related stuff

# hooks/network

Usually I like to have all my machines DNS names resolvable everywhere in my 
network, also the virtual ones. 
Normally, within a libvirt's network, this works quite well but 
only within this encapsulated environment. For other parts of my network these 
virtual host names are unknown, even for the host machine itself.

To solve this issue, I've realized the following solution:
* each virtual network will get its own DNS namespace: e.g. `virt.localnet`
* for each host machine, I start an additional _system_ dnsmasq instance
  (under Ubuntu this is easily possible by installing the `dnsmasq` package). 
  This _system_ dnsmasq instance will be used to forward DNS requests for
  `virt.localnet` to the desired libvirt's dnsmasq instance.
* the python script `hooks/network` is a libvirt network hook script, which will
  register a new libvirt network DNS domain at the system dnsmasq instance, iff
  the libvirt network definition meets certain conditions:
  * domains are currently only supported by `<forward>` mode `nat` or `routed`
  * the network must have an `<ip>` element defined
  * for the defined domain name, we require a specially _hacked_ format:
    `<domain>,<network/prefix>,local`. This has the following effect: since 
    libvirt will pass the value of the domain.name attribute directly to 
    dnsmasq's `domain` config option, we can instruct dnsmasq to use this 
    domain only for local resources and never send requests for this domain to 
    any upstream DNS servers. Otherwise we could easily create DNS request loops 
    ;) I think this would be a good standard behaviour for libvirt.
* my central DNS service (also dnsmasq running on a linksys router) is now 
  instructed to send requests for `virt.localnet` to the host system, which is 
  now able to chain this request to the libvirt's dnsmasq instance.
