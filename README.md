# adnix

A little script to convert ad blocking hosts files provided by Dan Pollock,
Peter Lowe, et al. into an Unbound or Dnsmasq configuration file.


## Introduction

If you want to block advertisements, spyware, malware and other unwanted
content you have a few choices:

* Install some add-ons in your favorite applications (so called 'Ad-blockers')
* Modify the /etc/hosts file on every client to redirect all hostname lookups
  to localhost
* Or make your site-local DNS server to do that for all clients

I don't use DNS servers of my ISP, instead I have an Unbound DNS server to
resolve all hostname lookups. This script downloads host lists maintained by
Dan Pollock, Peter Lowe and others that contain many hosts involved in
advertisement, spyware distribution, web trackers and other unwanted content.
Output of this script is a configuration file for Unbound, redirecting these
hostnames to localhost (you can change that if needed).


## Requirements

* unbound > 1.4.x
* Ruby 2.x


## Usage

This is a very short description of my own setup at home.

### Unbound

Configuration located in /etc/unbound. I added a subfolder /etc/unbound/conf.d/
and reduced /etc/unbound/unbound.conf to a bare minimum:

    server:
        include: "/etc/unbound/conf.d/*.conf"

The other configuration files are located in that conf.d subfolder, including
the file produced by this script.

### Dnsmasq

Option '--dnsmasq' produces ouput for Dnsmasq. 

    sudo adnix --dnsmasq --file /etc/dnsmasq.d/adnix

If you have NetworkManager start Dnsmasq:

    sudo adnix --dnsmasq --file /etc/NetworkManager/dnsmasq.d/adnix

Otherwise you have to 

### Periodic update

Create a cronjob for the update_unbound script and run it once every week or
month. I don't think the lists will change very often.

First unbound must be configured to accept unbound-control:

    sudo -u unbound unbound-control-setup

Edit /etc/unbound/unbound.conf and add these lines:

    remote-control:
      control-enable: yes

For cron:

    cp adnix.cron /etc/cron.d/adnix

For systemd:

    ln -s /usr/local/adnix/adnix.service /etc/systemd/system
    systemctl enable /usr/local/adnix/adnix.timer
    systemctl start adnix.timer

## Shell One-Liner

If you prefer a bash shell one-liner (only 2 sources, but you get the picture):

    { echo "server:" ; { curl -s 'http://pgl.yoyo.org/adservers/serverlist.php?hostformat=hosts&mimetype=plaintext'; curl -s 'http://someonewhocares.org/hosts/hosts'; } | sed s/#.*$// | awk '/^127\.0\.0\.1/ { print $2 }' | sort | uniq | awk '{ print "local-zone: \"" $1 "\" redirect\nlocal-data: \"" $1 " A 127.0.0.1\"\nlocal-data: \"" $1 " AAAA ::1\""}'; }


## References

* Script [unbound-block-hosts](https://github.com/jodrell/unbound-block-hosts) by Gavin Brown
* [Dan Pollock's list](http://someonewhocares.org/hosts/)
* [Peter Lowe's list](http://pgl.yoyo.org/adservers/)
* [Malware Domain List](http://www.malwaredomainlist.com/hostslist/hosts.txt)
