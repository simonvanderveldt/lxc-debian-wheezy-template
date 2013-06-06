LXC template lxc-debian-wheezy-template
=======================================

# Why
Since the lxc template for Debian in Debian Wheezy is broken (because it relies on live-debconfig which isn't available in Wheezy) I started looking for a solution.
Debian bug http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469 is the most relevant bug for this issue, though there are several other Debian bug's referenced on the LXC mailing list which actually don't solve much. This bug and Rob's template are referenced in several places, for example * Discussion: http://lists.alioth.debian.org/pipermail/freedombox-discuss/2013-February/005128.html.
In this Debian bug 680469 is a link from Rob van der Hoeven to a Debian Wheezy template he made by modifying the Debian Squeeze template. This template actually works in Wheezy, unlike the debconf template which is packaged in Wheezy's lxc (0.8.0~rc1-8+deb7u1). 
Unfortunately Debian lxc's package maintainer (Daniel Baumann) said the didn't want to support this working template (even though it barely differs from the upstream template) and then nothing happenend.
So now we have a non working Debian template in Debian's own stable lxc package.

# What
Anyway, I decided to try to get the template Rob created as close to upstream as possible so there is a working template anyone can use and which might have a chance of being added to a stable update.

Starting points were Rob's template, the last template from Debian's lxc package before the switch to the debconf template and the template from upstream.
* Rob's template can be found here http://freedomboxblog.nl/wp-content/uploads/lxc-debian-wheezy.gz, it's called lxc-debian-wheezy-robvdhoeven in this folder
* The last debian version with regular debian template instead of the debconf template is 0.7.5-5, the source of which is still available here https://launchpad.net/debian/sid/+source/lxc/0.7.5-5, it's called lxc-debian-0.7.5-5-debian in this folder
 * See changelog from 0.7.5-6 mentioning replacement of upstream debian template with debconf template https://launchpad.net/debian/sid/+source/lxc/0.7.5-6
* Upstream template from lxc 0.9.0 can be found here http://lxc.sourceforge.net/download/lxc/lxc-0.9.0.tar.gz, it's called lxc-debian-0.9.0-upstream in this folder

## Changes between Rob's template and Debian lxc 0.7.5-5's template
I first compared Rob's template with the template from Debian lxc 0.7.5-5. See debian-0.7.5-5_rob.diff for the diff. The differences between these are:
* Removed $SUITE (squeeze) and replaced it with harcoded wheezy
* Removed $MIRROR and replaced it with http://ftp.debian.org/debian
* Set default runlevel to 3
* Removed daemontools-run entry
* Added reporting of hostname to dhcp server
* Different way to set locale (same as upstream) (this uses locale-gen $LANG, which doesn't work on Wheezy (anymore?))
* Different pointless services to disable
 * Doesn't remove checkroot, umountroot, module-init-tools
* Changed random password for root to "root"
* Replacing the deprecated dhcp3-client package with isc-dhcp-client (same as upstream)
* Change arch-determination to simpler if structure without using dpkg or udpkg
* Added support for arch=armv5tel (results in arch=armel)
* Changed container configuration
 * Restructured it (all networking setting together)
 * Removed #lxc.console = /var/log/lxc/$name.console (was already commented out)
 * Removed lxc.cap.drop = sys_admin
 * Removed #lxc.cgroup.devices.allow = a (was already commented out)
 * Removed limits
 * Removed lxc.mount.entry for shared folder
 * Changed network settings
   * Removed lxc.network.mtu = 1500
   * Removed lxc.network.name = eth0
   * Removed lxc.network.veth.pair = veth-$name
   * Added lxc.network.ipv4 = 0.0.0.0/24
   * Replaced hard-coded mac address with random generated mac-address using new hex() function
* Added missing $name to copy_configuration() function call (same as upstream)

