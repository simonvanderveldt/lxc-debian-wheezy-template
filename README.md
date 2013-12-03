# LXC template lxc-debian-wheezy-template

## How to use
If you just want to get started with a working LXC template in Debian Wheezy read this section. Fore more background info about this repository and it's purpose see the chapters below.

Currently the working LXC template is a copy of the [template created by Rob van der Hoeven](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469#83).

### Install the template
* Download the file `lxc-debian-wheezy-robvdhoeven`
* Copy the file to the LXC templates folder `sudo cp lxc-debian-wheezy-robvdhoeven /usr/share/lxc/templates/lxc-debian-wheezy`
* Make root (group)owner `sudo chown root:root /usr/share/lxc/templates/lxc-debian-wheezy`
* Make the file executable `sudo chmod +x /usr/share/lxc/templates/lxc-debian-wheezy`

### Create a container using the template
`sudo lxc-create -n myfirstcontainer -t debian-wheezy`

The `-t` flag defines the template to use to create the container. It needs a template's short name to work. `debian-wheezy` is the short name of the `lxc-debian-wheezy` template we just added to our system. To get the short name of a template simply remove the `lxc-` prefix from the template's filename.

### Default configuration
The network is configured for DHCP using the `br0` interface. For more info see the [LXC SimpleBridge page on the Debian wiki](https://wiki.debian.org/LXC/SimpleBridge).
The default root password is `root`. Don't forget to change it!

Fore more information about LXC and Debian see the [LXC page on the Debian wiki](https://wiki.debian.org/LXC).


## Why did I create this repository?
The LXC template for Debian in Debian Wheezy is broken because it relies on live-debconfig which isn't available in Wheezy.

[Debian bug 680469](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469) is the most relevant bug for this issue, though there are several other Debian bug's referenced on the LXC mailing list which actually don't solve much.
In this Debian bug 680469 is a [link from Rob van der Hoeven](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469#83) to a Debian Wheezy template he made by modifying the Debian Squeeze template. This template actually works in Wheezy, unlike the debconf template which is packaged in Wheezy's LXC (0.8.0~rc1-8+deb7u1). 

Unfortunately this working template didn't make it into Debian Wheezy's release because LXC's package maintainer couldn't support something he didn't create. So now we have a non working Debian template in Debian's own stable LXC package.

With this repository I want to try to fix this.


## How to fix the situation
To try to fix this situation I decided to try to create a new template based on the last template from Debian's LXC package before the switch to debconf and add some of Rob's fixes to that. The primary goals are staying as close as possible to the Debian template so we have a chance that the fixed template get's added to a Debian stable update and offcourse making LXC work :)

As said, starting points were Rob's template, the last template from Debian's LXC package before the switch to the debconf template and the Debian template from [upstream](http://linuxcontainers.org/).
* Rob's template can be found on his site [freedomblox.nl](http://freedomboxblog.nl/wp-content/uploads/lxc-debian-wheezy.gz). The template file is called lxc-debian-wheezy-robvdhoeven in this repository.
* The last debian version with the regular debian template instead of the debconf template is 0.7.5-5, the source of which is still available on [launchpad](https://launchpad.net/debian/sid/+source/lxc/0.7.5-5). The template file is called lxc-debian-0.7.5-5-debian in this repository.
 * See changelog from [0.7.5-6](https://launchpad.net/debian/sid/+source/lxc/0.7.5-6) mentioning replacement of upstream debian template with debconf template 
* The latest upstream template from LXC (currently 0.9.0) can be found on the [Download page on the LXC Website](http://linuxcontainers.org/downloads/). The template file is called lxc-debian-0.9.0-upstream in this folder


### Changes between Rob's template and Debian LXC 0.7.5-5's template
Below follows a list of the changes in Rob's template compared to the template from Debian's LXC 0.7.5-5 package with some info on the changes and if the changes should be kept or not.

* Removed $SUITE (squeeze) and replaced it with harcoded wheezy. KEEP hardcoded wheezy for now
* Removed $MIRROR and replaced it with http://ftp.debian.org/debian. RE-ADD mirror without variable, should probably use http.debian.net, see the [Debian GeoMirror page on the Debian wiki](http://wiki.debian.org/DebianGeoMirror)
* Set default runlevel to 3, was added to [0.7.4.2-4](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-4), but somehow not included in the releases after that. Also included in upstream. [Default in Debian is 2](https://wiki.debian.org/RunLevel) KEEP runlevel 3
* Removed adding of daemontools-run inittab entry, was added in [0.7.5-4](https://launchpad.net/debian/sid/+source/lxc/0.7.5-4), not in upstream, I see no direct need to re-add it. KEEP removed
* Added reporting of hostname to dhcp server. I see no direct need for this for everybody. REMOVE
* Different way to set locale (same as upstream), this uses locale-gen $LANG, which doesn't work on Wheezy (anymore?). FIX needed
* Disables less pointless services
 * checkroot, was added in [0.7.3-1](https://launchpad.net/debian/wheezy/+source/lxc/0.7.3-1) as a fix for [Debian bug 601001](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=601001) and upstreamed in [0.7.4.2-0.1](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-0.1). RE-ADD disable checkroot
 * umountroot, was added in [0.7.4.2-0.1](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-0.1) and re-added in [0.7.4.2-4](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-4) as a fix for [Debian bug 611972](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=611972). RE-ADD disable umountroot
 * module-init-tools, was added in [0.7.4.2-1](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-1). RE-ADD disable module-init-tools
* Changed random password for root to "root". Easier to remember than random password and there is a message notifying the user of this standard password, KEEP
* Replacing deprecated dhcp3-client package with isc-dhcp-client (same as upstream). KEEP
* Change arch-determination to simpler if structure without using dpkg or udpkg. Seems to work and is simpler, so KEEP
* Added support for arch=armv5tel (results in arch=armel). Does no harm as far as I can see, so KEEP
* Changed container configuration
  * Restructured it (all networking setting together) KEEP
  * Removed #lxc.console = /var/log/lxc/$name.console (was already commented out). KEEP removed
  * Removed lxc.cap.drop = sys_admin. Drops CAP_SYS_ADMIN capabilities. As far as I could find out it's fine to remove this, so KEEP removed
  * Removed #lxc.cgroup.devices.allow = a (was already commented out). KEEP removed
  * Removed limits. Limits aren't in upstream either, so KEEP removed
  * Removed lxc.mount.entry for shared folder. Should be in config file, so KEEP removed
  * Changed network settings
    * Removed lxc.network.mtu = 1500. Isn't necessary, so KEEP removed
    * Removed lxc.network.name = eth0. Isn't necessary, so KEEP removed
    * Removed lxc.network.veth.pair = veth-$name. Isn't necessary, so KEEP removed.
    * Added lxc.network.ipv4 = 0.0.0.0/24. Makes it possible to use DHCP with the random-generated MAC address. KEEP
    * Replaced hard-coded mac address with random generated mac-address using new hex() function. KEEP
* Added missing $name to copy_configuration() function call (same as upstream). KEEP
