LXC template lxc-debian-wheezy-template
=======================================

# How
Download the file `lxc-debian-wheezy-robvdhoeven`
```
sudo cp lxc-debian-wheezy-robvdhoeven /usr/share/lxc/templates/lxc-debian-wheezy
sudo chmod +x /usr/share/lxc/templates/lxc-debian-wheezy
name=container-name
sudo lxc-create -n $name -t debian-wheezy
```
Put the name you want for your container in place of `container-name`.
root password is root. More information at https://wiki.debian.org/LXC

The network is configured for DHCP. You can change it at `/var/lib/lxc/$name/rootfs/etc/network/interfaces`.
See https://wiki.debian.org/LXC/SimpleBridge

# Why
Since the lxc template for Debian in Debian Wheezy is broken (because it relies on live-debconfig which isn't available in Wheezy) I started looking for a solution.

[Debian bug 680469](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469) is the most relevant bug for this issue, though there are several other Debian bug's referenced on the LXC mailing list which actually don't solve much.
In this Debian bug 680469 is a [link from Rob van der Hoeven](http://bugs.debian.org/cgi-bin/bugreport.cgi?bug=680469#83) to a Debian Wheezy template he made by modifying the Debian Squeeze template. This template actually works in Wheezy, unlike the debconf template which is packaged in Wheezy's lxc (0.8.0~rc1-8+deb7u1). 

Unfortunately Debian lxc's package maintainer (Daniel Baumann) said the didn't want to support this working template (even though it barely differs from the upstream template) and then nothing happenend.
So now we have a non working Debian template in Debian's own stable lxc package.

# What
Anyway, I decided to try to get the template Rob created as close to upstream as possible so there is a working template anyone can use and which might have a chance of being added to a stable update.

Starting points were Rob's template, the last template from Debian's lxc package before the switch to the debconf template and the template from upstream.
* Rob's template can be found here http://freedomboxblog.nl/wp-content/uploads/lxc-debian-wheezy.gz. The file is called lxc-debian-wheezy-robvdhoeven in this folder
* The last debian version with the regular debian template instead of the debconf template is 0.7.5-5, the source of which is still available here https://launchpad.net/debian/sid/+source/lxc/0.7.5-5. The file template file is called lxc-debian-0.7.5-5-debian in this folder
 * See changelog from [0.7.5-6](https://launchpad.net/debian/sid/+source/lxc/0.7.5-6) mentioning replacement of upstream debian template with debconf template 
* The upstream template from lxc 0.9.0 can be found here http://lxc.sourceforge.net/download/lxc/lxc-0.9.0.tar.gz. The file is called lxc-debian-0.9.0-upstream in this folder

## Changes between Rob's template and Debian lxc 0.7.5-5's template
I first compared Rob's template with the template from Debian lxc 0.7.5-5. See debian-0.7.5-5_rob.diff for the diff. Below is a list of the differences, info about them and if the change should be kept or reversed.
* Removed $SUITE (squeeze) and replaced it with harcoded wheezy. KEEP hardcoded wheezy for now
* Removed $MIRROR and replaced it with http://ftp.debian.org/debian. RE-ADD mirror without variable, should probably use http.debian.net, see http://wiki.debian.org/DebianGeoMirror
* Set default runlevel to 3, was added to [0.7.4.2-4](https://launchpad.net/debian/sid/+source/lxc/0.7.4.2-4), but somehow not included in the releases after that. Also included in upstream. KEEP runlevel 3
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

