Onix Seeder
===========

Onix-seeder is a crawler for the Onix blockchain, which exposes a list
of reliable nodes via a built-in DNS server.

Features:
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes down to v1.2.0 to request new IP addresses from,
  but only reports good post-v1.2.0 nodes.
* keeps statistics over (exponential) windows of 2 hours, 8 hours,
  1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default 24 threads simultaneously).

REQUIREMENTS
------------

    $ sudo apt-get install build-essential libboost-all-dev libssl-dev

USAGE
-----

Assuming you want to run a dns seed on `dnsseed.example.com`, you will
need an authorative NS record in example.com's domain record, pointing
to for example `vps.example.com`:

    $ dig -t NS dnsseed.example.com
    
    ;; ANSWER SECTION
    dnsseed.example.com.   86400    IN      NS     vps.example.com.

On the system vps.example.com, you can now run dnsseed:

    $ ./dnsseed -h dnsseed.example.com -n vps.example.com

If you want the DNS server to report SOA records, please provide an
e-mail address (with the @ part replaced by .) using -m.

NOTE FOR UBUNTU
---------------

If you get timeouts when testing your setup, you need to disable the `systemd-resolved` service with the following cmds:

    $ sudo systemctl stop systemd-resolved
    $ sudo systemctl disable systemd-resolved

ADDITIONAL SETUP
----------------

The seeders periodically scan for two files "nodes4.txt" (ipv4); "nodes6.txt" (ipv6). (Plain text files with one ip per line - no port numbering)
If these files are present, the seeders will use these as an additional source of IPs to feed into the crawler, this can help get better network coverage.

For best seeder performance, it is advised to acquire an external source that can provide such a list of IPs and then use a cronjob to have it periodically place new versions of the lists in these files.

Example of command to run for a node running on the same machine and directory where the seeder is running:

    $ onix-cli getpeerinfo | grep '"addr":' | \  
          sed -E "s/ +\"addr\": \"//g" | sed -E "s/\",//g" | grep ':5888' | sed -E "s/:5888//g" \
             > nodes4.txt

That, assuming that the seeder is located into the seeder directory straight in the onixd path.

COMPILING
---------
Compiling will require boost and ssl.  On debian systems, these are provided
by `libboost-dev` and `libssl-dev` respectively.

    $ make

This will produce the `dnsseed` binary.

RUNNING AS NON-ROOT
-------------------

Typically, you'll need root privileges to listen to port 53 (name service).

One solution is using an iptables rule (Linux only) to redirect it to
a non-privileged port:

    $ sudo iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 15353

If properly configured, this will allow you to run dnsseed in userspace, using
the -p 5353 option.

Another solution is allowing a binary to bind to ports < 1024 with setcap (IPv6 access-safe)

    $ setcap 'cap_net_bind_service=+ep' /path/to/dnsseed

CREDITS
-------

This software is a fork of [Bitcoin Seeder](https://github.com/sipa/bitcoin-seeder) by Pieter Wuille.

Customization instructions taken from 
[How to create a DNS seeder for your new alt-coin](https://motion-software.com/blog/how-to-create-a-dns-seeder-for-your-new-alt-coin/) 
by Aleksandar Aramov.
