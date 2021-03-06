# Mr. Netops's nifty inline resolver

If you have ever needed to resolve hostnames or ip address in arbitrarily formatted data, **nir** is the script for you.

**nir** will detect the following types of values

* Hostnames
* IPv4 Addresses
* IPv6 Addresses

in arbitrarily formatted data from standard in and will perform inline resolution including

* Hostnames
* IPv4 Addresses
* IPv6 Addresses
* [getaddrinfo canonname](http://linux.die.net/man/3/getaddrinfo)

Some trivial examples

```bash
$ echo "fi fi fo www.google.com fum" | nir
fi fi fo www.google.com[216.58.216.36,2607:f8b0:4007:805::1012] fum

$ echo "www.google.com216.58.216.36" | nir
www.google.com[216.58.216.4,2607:f8b0:4007:809::2004]216.58.216.36[lax02s22-in-f4.1e100.net]
```

How about something a little more fun?
```bash
$ dig facebook.com ANY | nir

; <<>> DiG 9.9.5-3ubuntu0.2-Ubuntu <<>> facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1] ANY
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 37696
;; flags: qr rd ra; QUERY: 1, ANSWER: 7, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].			IN	ANY

;; ANSWER SECTION:
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		21578	IN	NS	a.ns.facebook.com[69.171.239.12].
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		21578	IN	NS	b.ns.facebook.com[69.171.255.12].
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		21578	IN	TXT	"v=spf1 redirect=_spf.facebook.com"
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		278	IN	MX	10 msgin.vvv.facebook.com[173.252.113.23].
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		98	IN	SOA	a.ns.facebook.com[69.171.239.12]. dns.facebook.com[31.13.77.6,2a03:2880:f022:6:face:b00c:0:2,star.c10r.facebook.com]. 1426982281 7200 1800 604800 120
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		878	IN	A	173.252.120.6
facebook.com[173.252.120.6,2a03:2880:2130:cf05:face:b00c:0:1].		878	IN	AAAA	2a03:2880:2130:cf05:face:b00c:0:1

;; Query time: 43 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Mar 21 17:00:32 PDT 2015
;; MSG SIZE  rcvd: 232
```

Ever wondered if your ip addresses and hostnames are really going where you think they are?

Great for checking 

* Firewall configs
* Load balancer configs
* Any kind of network config really
* Dns zone files
* Application configs
* Arp output
* /etc/hosts

The list goes on.

For example
```markdown
cat someConfig | nir
pool public {
      ipAddress1[hostname1]
      ipAddress2[**wrongHostname**]
      ipAddress3[hostname3]
}

object-group network host_name_4
 network-object host ipAddress4[**wrongHostname**]
```
or
```markdown
cat someConfig | nir
pool public {
      hostname1[ipAddress1]
      hostname2[**wrongSubnetIp**]
      hostname3[ipAddress3]
}
```

You can chain nir to inspect forward and reverse lookup

```bash
$ echo "www.amazon.com" | nir | nir
www.amazon.com[176.32.98.166][205.251.242.103[s3-console-us-standard.console.aws.amazon.com]]
```

Usage information from **nir --help**

```
Mr. Netops's nifty inline resolver

Options:

	-format <format>
		Specify alternate formatting for resolved values
	-separator <separator>
		Specify alternate separator for lookups with multiple values
	-tags
		Add type tags to resolved values
	-verbose
		Print additional status information to stderr
	-debug
		Print debugging information to stderr
	-timeout <timeout>
		Dns lookup timeout
	-lookup <types>
		Types of resolved values and their presentation order
		Supports The following types: hostname, ipv4, ipv6, canonname
	-help
		This

Advanced Options:

	-hostnameRegexp <regexp>
		Specify an alternate hostname regular expression
	-ipV4AddressRegexp <regexp>
		Specify an alternate ipv4 address regular expression
	-ipV6AddressRegexp <regexp>
		Specify an alternate ipv6 address regular expression
	-ipV6Support
		Manually enable ipv6 support
	-noipV6Support
		Manually disable ipv6 support

Examples:

	Basic invocation
		someCommand | nir

	Present ipv4 & ipv6 resolved values in that order
		someCommand | nir -lookup ipv4,ipv6

	Format inline lookups in curly braces and seperate values with ;
		someCommand | nir -format '{%s}' -separator ';'

Use Cases:
			
	dig facebook.com ANY | nir
	mtr -wr facebook.com | nir
	cat /etc/hosts | nir
	arp -a | nir
```	

