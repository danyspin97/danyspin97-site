---
title: "Security and privacy on UNIX desktop"
date: 2019-08-13T16:30:00+02:00
category: ["security", "privacy"]
tag: ["stubby", "dns", "dnsmasq", "hblock"]
hacker_news_id: "20686166"
---

Security and privacy are two important factors of the IT field. A lot of effort
has been made to ensure safe surfing on the net, whatever we are connected to
our home router, the university network or even worse the airport public
network. And other effort have been made to avoid malicious sites.

Yet, systems don't offer privacy and security out of the box.
This post will guide you to the installation of some tools that will improve
your security, privacy and performance on a UNIX system.

## hblock [^1]

The first thing suggested in every post about security and privacy is to
install web browser extensions to block ads and malicous sites.

What if instead we could block requests at a system level? We can map every
site we want to avoid with the address `0.0.0.0` in `/etc/hosts`: every time
the browser or *any* application on the system request one of these sites, the
above address will be used, aborting the connection.

Little explanation about how this works:

: The domain names, such as `www.example.com`, have to be translated into
internet addresses and it's the nameserver (or **DNS** server) responsability
to do so. `/etc/hosts` is a file that maps domains to addresses without calling
the nameserver: adding `8.8.8.8 dns.google.com` will map `dns.google.com` to
the address `8.8.8.8`.

So far, so good. But do we have to add every site by hand? `hblock` is a little
script that will do so for us. It collect malicious sites from different sources
[^2] and write them to `/etc/hosts`.

To install, run make install as root inside the repository:

```bash
$ git clone https://github.com/hectorm/hblock.git
$ cd hblock
$ sudo make install
```

Now, to run the program just call `hblock`.

If your system is powered by systemd, you can enable `hblock` timer so that
`/etc/hosts` will be updated daily.

Its site [^1] also have auto-generated files for different programs (including
one that will be introduced later in this post) if you don't want to download
and install the script.

## stubby [^6]

The nameserver itself is a source of attack and privacy leaks. The DNS
protocol, which has been originally specified in 1983 [^4], transmit domain
name queries in clear text. It also means that everyone can snoop this data
and collect information about *which* sites you have visited. Also, queries are
not verified. For more information, checkout “*Why is DNS a privacy concern?* ”
[^7].

DNSSEC [^5] is the solution to make DNS more secure but it does not improve
privacy and is not enabled by default.

Further attempts have been made and are still in progress to improve DNS.
The most important ones [^8] are:

- dnscrypt-proxy [^9]
- DNS Over TLS (*DoT*)
- DNS Over HTTPS (*DoH*)

In this post, we will use *DoT* as it is a standardized protocol with working
implementations [^10] and Stubby, a stub resolver.

Install stubby using the package manager. The default configuration will be
installed into `/etc/stubby/stubby.yml` and it contains sane values; the only
thing we need to change is the port stubby listens to:

```
listen_addresses:
  - 127.0.0.1@53000
  - 0::1@53000
```

Make sure that `listen_addresses` config matches the above snippet.

The nameserver used are hard-coded into the config; to enable/disable them,
comment the relevant lines.

Run stubby using systemd service or the service manager installed currently
installed on your system.

```bash
$ systemctl enable stubby
$ systemctl start stubby
```

At this point we have a stub resolver listening on localhost, port 53000.
However, each query can take from 200ms to 500ms to be resolved, whereas DNS in
clear text usually takes only ~50ms. That's a huge slowdown; in the next
section we will partially avoid all this overhead by using `dnsmasq`, a local
server used to cache DNS queries.

**Note:** *One caveat of stubby currently is the RAM usage, which can go up to
200MB.*

## dnsmasq [^10]

dnsmasq is both a DHCP and a DNS server; we will only enable the DNS server
capability so that it can be used as a local cache server.

Ensure that the configuration located at `/etc/dnsmasq.conf` contains the
following values:

```
# Do not forward plain names
domain-needed
# Do not forward invalid address spaces
bogus-priv

# Do not use /etc/resolv file
no-resolv

# Point the nameserver to stubby
server=::1#53000
server=127.0.0.1#53000

# Listen only on the loopback address
listen-address=::1,127.0.0.1

# Disable DHCP server capabilities
no-dhcp-interface

# Increase the cache size
cache-size=1000
```

Start dnsmasq with systemd or any other service manager:

```bash
$ systemctl enable dnsmasq
$ systemctl start dnsmasq
```

Now, to make the system use dnsmasq, the file `/etc/resolv.conf` (which
contains the list of the nameserver addresses currently known) must be
exactly as this:

```
nameserver 127.0.0.1
```

***Note:*** *If no port is specified, as in the above snippet, the default port
53 will be used*.

However, some network managers (like *NetworkManager*) rewrite
`/etc/resolv.conf`, overwriting our choice to use dnsmasq. In this case
*openresolve* [^11] must be used: write the following into the file
`/etc/resolvconf.conf`:

```
# Use the local name server
name_servers="::1 127.0.0.1"
```

and then run:

```bash
$ sudo resolvconf -u
```

How does this work? dnsmasq will be used as the default nameserver, dispatching
every DNS query to stubby. Every query resolved will be cached locally, so that
the next time dnsmasq will already have the response. The time to translate
a specific domain name to an internet address (*IP*) will be high for the first
query (when stubby actually ask a remote nameserver) and instant for the
later ones.

To test if it is working as intended run `drill` (usually contained in a
`ldns-utils` package if it is not already installed):

```bash
$ drill exherbo.org
;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: xxxxx
;; flags: qr rd ra ; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0 
;; QUESTION SECTION:
;; exherbo.org.	IN	A

;; ANSWER SECTION:
exherbo.org.	360	IN	A	185.38.172.64

;; AUTHORITY SECTION:

;; ADDITIONAL SECTION:

;; Query time: 321 msec
;; EDNS: version 0; flags: ; udp: 4096
;; SERVER: 127.0.0.1
;; WHEN: Sun Aug 11 18:25:59 2019
;; MSG SIZE  rcvd: 67
```

The `SERVER` used is indeed dnsmasq (listening on `localhost` or `127.0.0.1`)
and stubby is being used accordingly to the high value of the query time.

Let's call it again to test dnsmasq cache:

```bash
$ drill exherbo.org
;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: 23991
;; flags: qr rd ra ; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0 
;; QUESTION SECTION:
;; exherbo.org.	IN	A

;; ANSWER SECTION:
exherbo.org.	340	IN	A	185.38.172.64

;; AUTHORITY SECTION:

;; ADDITIONAL SECTION:

;; Query time: 0 msec
;; SERVER: 127.0.0.1
;; WHEN: Sun Aug 11 18:26:20 2019
;; MSG SIZE  rcvd: 45
```

As you can see, the query time this time is 0 millisecond; the cache is indeed
working!

## Conclusion

For this post is all, folks! If you know any software or suggestion you would
like to see in this guide, just write me an email or send a Pull Request to
this repository [^12]!

[^1]: https://hblock.molinero.dev/
[^2]: https://github.com/hectorm/hblock#sources
[^3]: https://raw.githubusercontent.com/hectorm/hblock/master/hblock
[^4]: https://en.wikipedia.org/wiki/Domain_Name_System#cite_note-19
[^5]: https://www.dnssec.net/
[^6]: https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+Daemon+-+Stubby
[^7]: https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+-+The+Problem
[^8]: https://dnsprivacy.org/wiki/display/DP/DNS+Privacy+-+The+Solutions
[^9]: https://dnscrypt.info/
[^10]: http://www.thekelleys.org.uk/dnsmasq/doc.html
[^11]: https://wiki.archlinux.org/index.php/Openresolv
[^12]: https://github.com/danyspin97/danyspin97-site
