
# Table of Contents

1.  [INTRODUCTION](#orgf5d5f91)
2.  [PASSIVE Information Gathering](#orgf0c581c)
    1.  [WHOIS](#orgc5799f6)
    2.  [DNS](#org181247e)
        1.  [Nslookup \\ Dig](#org1cbc42a)
    3.  [Passive Subdomain Enumeration](#orgaa4c0fa)
        1.  [VirusTotal](#org02a381d)
        2.  [Certificates](#orgc620eb2)
        3.  [Automating Passive Subdomain Enumeration](#org56e9529)
    4.  [Passive Infrastructure Identification](#org32e4a8b)
3.  [ACTIVE Informatin Gathering](#org26482b8)
    1.  [Active Infrastructure Identification](#orgc60b326)
        1.  [HTTP Headers](#orgc4704bb)
        2.  [WhatWeb](#orgf903f12)
        3.  [WafW00f](#org74c1e0e)
        4.  [Acquatone](#orgc958118)
    2.  [Active Subdomain Enum](#org763f317)
        1.  [ZoneTrasfer](#org1d05a6e)
        2.  [Gobuster](#org802df77)
    3.  [Virtual Hosts](#org4a46c12)
        1.  [IP-based Virtual Hosting](#org3cb5308)
        2.  [Name-based Virtual Hosting](#orgde50425)
        3.  [Automating Virtual Hosts Discovery](#orgc46945f)
    4.  [Crawling](#orgd4ab596)
        1.  [FFuF](#org7d47081)
        2.  [Sensitive Information Disclosure](#org6a9e611)



<a id="orgf5d5f91"></a>

# INTRODUCTION


<a id="orgf0c581c"></a>

# PASSIVE Information Gathering


<a id="orgc5799f6"></a>

## WHOIS

**Whois** is a `TCP-based` transaction-oriented query/response protocol listening on TCP port <span class="underline">43</span> by default. We can use it for <span class="underline">querying databases</span> containing domain names, <span class="underline">IP addresses</span>, or <span class="underline">autonomous systems</span> and provide information services to Internet users.

`In simple terms, the Whois database is a searchable list of all domains currently registered worldwide.`

`whois` command is:

    export TARGET="facebook.com" # Assign our target to an environment variable
    whois $TARGET
    
    Domain Name: FACEBOOK.COM
    Registry Domain ID: 2320948_DOMAIN_COM-VRSN
    Registrar WHOIS Server: whois.registrarsafe.com
    Registrar URL: https://www.registrarsafe.com
    Updated Date: 2021-09-22T19:33:41Z
    Creation Date: 1997-03-29T05:00:00Z
    Registrar Registration Expiration Date: 2030-03-30T04:00:00Z
    Registrar: RegistrarSafe, LLC
    Registrar IANA ID: 3237
    Registrar Abuse Contact Email: abusecomplaints@registrarsafe.com
    Registrar Abuse Contact Phone: +1.6503087004
    Domain Status: clientDeleteProhibited https://www.icann.org/epp#clientDeleteProhibited
    Domain Status: clientTransferProhibited https://www.icann.org/epp#clientTransferProhibited
    Domain Status: clientUpdateProhibited https://www.icann.org/epp#clientUpdateProhibited
    Domain Status: serverDeleteProhibited https://www.icann.org/epp#serverDeleteProhibited
    Domain Status: serverTransferProhibited https://www.icann.org/epp#serverTransferProhibited
    Domain Status: serverUpdateProhibited https://www.icann.org/epp#serverUpdateProhibited
    Registry Registrant ID:
    Registrant Name: Domain Admin
    
    <Snip>

We can gather the same data using `whois.exe` from <span class="underline">Windows Sysinternals</span>:

    whois.exe facebook.com
    
    Whois v1.21 - Domain information lookup
    Copyright (C) 2005-2019 Mark Russinovich
    Sysinternals - www.sysinternals.com
    
    Connecting to COM.whois-servers.net...
    
    <Snip>


<a id="org181247e"></a>

## DNS

DNS converts domain names to IP addresses, allowing browsers to access resources on the Internet.

`Resource Record` =>	A domain name, usually a fully qualified domain name, is the first part of a Resource Record. If you don't use a fully qualified domain name, the zone's name where the record is located will be appended to the end of the name.
`TTL` => 	In seconds, the Time-To-Live (`TTL`) defaults to the minimum value specified in the SOA record.
`Record Class` =>	Internet, Hesiod, or Chaos
`Start Of Authority` (`SOA`) =>	It should be first in a zone file because it indicates the start of a zone. Each zone can only have one `SOA` record, and additionally, it contains the zone's values, such as a serial number and multiple expiration timeouts.
`Name Servers` (`NS`) => 	The distributed database is bound together by `NS` Records. They are in charge of a zone's authoritative name server and the authority for a child zone to a name server.
`IPv4 Addresses` (`A`) => 	The A record is only a mapping between a hostname and an IP address. 'Forward' zones are those with `A` records.
`Pointer` (`PTR`) => 	The PTR record is a mapping between an IP address and a hostname. 'Reverse' zones are those that have `PTR` records.
`Canonical Name` (`CNAME`) => 	An alias hostname is mapped to an `A` record hostname using the `CNAME` record.
`Mail Exchange` (`MX`) => 	The `MX` record identifies a host that will accept emails for a specific host. A priority value has been assigned to the specified host. Multiple MX records can exist on the same host, and a prioritized list is made consisting of the records for a specific host.


<a id="org1cbc42a"></a>

### Nslookup \\ Dig

With `Nslookup`, we can search for domain name servers on the Internet and ask them for information about hosts and domains.

-   Querying: `A` Records
    
        export TARGET="facebook.com"
        nslookup $TARGET
        
        Server:		1.1.1.1
        Address:	1.1.1.1#53
        
        Non-authoritative answer:
        Name:	facebook.com
        Address: 31.13.92.36
        Name:	facebook.com
        Address: 2a03:2880:f11c:8083:face:b00c:0:25de
    
    Unlike nslookup, `DIG` shows us some more information that can be of
    importance.
    
        dig facebook.com @1.1.1.1
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> facebook.com @1.1.1.1
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58899
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 1232
        ;; QUESTION SECTION:
        ;facebook.com.                  IN      A
        
        ;; ANSWER SECTION:
        facebook.com.           169     IN      A       31.13.92.36
        
        <Snip>
-   Querying: `A` Records for a `Subdomain`
    
        export TARGET=www.facebook.com
        nslookup -query=A $TARGET
        
        Server:		1.1.1.1
        Address:	1.1.1.1#53
        
        Non-authoritative answer:
        www.facebook.com	canonical name = star-mini.c10r.facebook.com.
        Name:	star-mini.c10r.facebook.com
        Address: 31.13.92.36
    
        dig a www.facebook.com @1.1.1.1
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> a www.facebook.com @1.1.1.1
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15596
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 1232
        ;; QUESTION SECTION:
        ;www.facebook.com.              IN      A
        
        ;; ANSWER SECTION:
        www.facebook.com.       3585    IN      CNAME   star-mini.c10r.facebook.com.
        star-mini.c10r.facebook.com. 45 IN      A       31.13.92.
-   Querying: `PTR` Records for an IP Address
    
        nslookup -query=PTR 31.13.92.36
        
        Server:		1.1.1.1
        Address:	1.1.1.1#53
        
        Non-authoritative answer:
        36.92.13.31.in-addr.arpa	name = edge-star-mini-shv-01-frt3.facebook.com.
        
        Authoritative answers can be found from:
    
        dig -x 31.13.92.36 @1.1.1.1
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> -x 31.13.92.36 @1.1.1.1
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 51730
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 1232
        ;; QUESTION SECTION:
        ;36.92.13.31.in-addr.arpa.      IN      PTR
        
        ;; ANSWER SECTION:
        36.92.13.31.in-addr.arpa. 1028  IN      PTR     edge-star-mini-shv-01-frt3.facebook.com.
-   Querying: `ANY` Existing Records
    
            export TARGET="google.com"
        nslookup -query=ANY $TARGET
        
        Server:		10.100.0.1
        Address:	10.100.0.1#53
        
        Non-authoritative answer:
        Name:	google.com
        Address: 172.217.16.142
        Name:	google.com
        Address: 2a00:1450:4001:808::200e
        google.com	text = "docusign=05958488-4752-4ef2-95eb-aa7ba8a3bd0e"
        google.com	text = "docusign=1b0a6754-49b1-4db5-8540-d2c12664b289"
        google.com	text = "v=spf1 include:_spf.google.com ~all"
        google.com	text = "MS=E4A68B9AB2BB9670BCE15412F62916164C0B20BB"
        google.com	text = "globalsign-smime-dv=CDYX+XFHUw2wml6/Gb8+59BsH31KzUr6c1l2BPvqKX8="
        google.com	text = "apple-domain-verification=30afIBcvSuDV2PLX"
        google.com	text = "google-site-verification=wD8N7i1JTNTkezJ49swvWW48f8_9xveREV4oB-0Hf5o"
        google.com	text = "facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
        google.com	text = "google-site-verification=TV9-DBe4R80X4v0M4U_bd_J9cpOJM0nikft0jAgjmsQ"
        google.com	nameserver = ns3.google.com.
        google.com	nameserver = ns2.google.com.
        google.com	nameserver = ns1.google.com.
        google.com	nameserver = ns4.google.com.
        google.com	mail exchanger = 10 aspmx.l.google.com.
        google.com	mail exchanger = 40 alt3.aspmx.l.google.com.
        google.com	mail exchanger = 20 alt1.aspmx.l.google.com.
        google.com	mail exchanger = 30 alt2.aspmx.l.google.com.
        google.com	mail exchanger = 50 alt4.aspmx.l.google.com.
        google.com
        
        <Snip>
    
        dig any google.com @8.8.8.8
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> any google.com @8.8.8.8
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 49154
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 22, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 512
        ;; QUESTION SECTION:
        ;google.com.                    IN      ANY
        
        ;; ANSWER SECTION:
        google.com.             249     IN      A       142.250.184.206
        google.com.             249     IN      AAAA    2a00:1450:4001:830::200e
        google.com.             549     IN      MX      10 aspmx.l.google.com.
        google.com.             3549    IN      TXT     "apple-domain-verification=30afIBcvSuDV2PLX"
        google.com.             3549    IN      TXT     "facebook-domain-verification=22rm551cu4k0ab0bxsw536tlds4h95"
        google.com.             549     IN      MX      20 alt1.aspmx.l.google.com.
        google.com.             3549    IN      TXT     "docusign=1b0a6754-49b1-4db5-8540-d2c12664b289"
        google.com.             3549    IN      TXT     "v=spf1 include:_spf.google.com ~all"
        google.com.             3549    IN      TXT     "globalsign-smime-dv=CDYX+XFHUw2wml6/Gb8+59BsH31KzUr6c1l2BPvqKX8="
        google.com.             3549    IN      TXT     "google-site-verification=wD8N7i1JTNTkezJ49swvWW48f8_9xveREV4oB-0Hf5o"
        google.com.             9       IN      SOA     ns1.google.com. dns-admin.google.com. 403730046 900 900 1800 60
        google.com.             21549   IN      NS      ns1.google.com.
        google.com.             21549   IN      NS      ns3.google.com.
        
        <Snip>
-   Querying: `TXT` Records
    
            export TARGET="facebook.com"
        nslookup -query=TXT $TARGET
        
        Server:		1.1.1.1
        Address:	1.1.1.1#53
        
        Non-authoritative answer:
        facebook.com	text = "v=spf1 redirect=_spf.facebook.com"
        facebook.com	text = "google-site-verification=A2WZWCNQHrGV_TWwKh6KHY90tY0SHZo_RnyMJoDaG0s"
        facebook.com	text = "google-site-verification=wdH5DTJTc9AYNwVunSVFeK0hYDGUIEOGb-RReU6pJlY"
    
        dig txt facebook.com @1.1.1.1
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> txt facebook.com @1.1.1.1
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 63771
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 1232
        ;; QUESTION SECTION:
        ;facebook.com.                  IN      TXT
        
        ;; ANSWER SECTION:
        facebook.com.           86400   IN      TXT     "v=spf1 redirect=_spf.facebook.com"
        facebook.com.           7200    IN      TXT     "google-site-verification=A2WZWCNQHrGV_TWwKh6KHY90tY0SHZo_RnyMJoDaG0s"
        facebook.com.           7200    IN      TXT     "google-site-verification=wdH5DTJTc9AYNwVunSVFeK0hYDGUIEOGb-RReU6pJlY"
        
        <Snip>
-   Querying: `MX` Records
    
        export TARGET="facebook.com"
        nslookup -query=MX $TARGET
        
        Server:		1.1.1.1
        Address:	1.1.1.1#53
        
        Non-authoritative answer:
        facebook.com	mail exchanger = 10 smtpin.vvv.facebook.com.
        
        Authoritative answers can be found from:
    
        dig mx facebook.com @1.1.1.1
        
        ; <<>> DiG 9.16.1-Ubuntu <<>> mx facebook.com @1.1.1.1
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 9392
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
        
        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 1232
        ;; QUESTION SECTION:
        ;facebook.com.                  IN      MX
        
        ;; ANSWER SECTION:
        facebook.com.           3600    IN      MX      10 smtpin.vvv.facebook.com.
        
        <Snip>

So far, we have gathered `A`, `NS`, `MX`, and `CNAME` records with the nslookup and dig commands. Organizations are given IP addresses on the Internet, but they aren't always their owners. They might rely on `ISPs` and hosting provides that lease smaller netblocks to them.

We can combine some of the results gathered via nslookup with the whois database to determine if our target organization uses hosting providers. This combination looks like the following example:
<span class="underline">Nslookup</span>

    export TARGET="facebook.com"
    nslookup $TARGET
    
    Server:		1.1.1.1
    Address:	1.1.1.1#53
    
    Non-authoritative answer:
    Name:	facebook.com
    Address: 157.240.199.35
    Name:	facebook.com
    Address: 2a03:2880:f15e:83:face:b00c:0:25de

<span class="underline">WHOIS</span>

    whois 157.240.199.35
    
    NetRange:       157.240.0.0 - 157.240.255.255
    CIDR:           157.240.0.0/16
    NetName:        THEFA-3
    NetHandle:      NET-157-240-0-0-1
    Parent:         NET157 (NET-157-0-0-0-0)
    NetType:        Direct Assignment
    OriginAS:
    Organization:   Facebook, Inc. (THEFA-3)
    RegDate:        2015-05-14
    Updated:        2015-05-14
    Ref:            https://rdap.arin.net/registry/ip/157.240.0.0
    
    <Snip>


<a id="orgaa4c0fa"></a>

## Passive Subdomain Enumeration

Exploring `subdomains` in a domain name to uncover hidden areas, <span class="underline">expanding attack possibilities</span> through passive enumeration initially, with plans for more active methods later.

At this point, we will only perform passive subdomain enumeration using third-party services or publicly available information.


<a id="org02a381d"></a>

### VirusTotal

VirusTotal maintains its DNS replication service, which is developed by preserving DNS resolutions made when users visit URLs given by them. To receive information about a domain, type the domain name into the search bar and click on the "Relations" tab.


<a id="orgc620eb2"></a>

### Certificates

Another interesting source of information we can use to extract subdomains is SSL/TLS certificates. The main reason is Certificate Transparency (CT), a project that requires every SSL/TLS certificate issued by a Certificate Authority (CA) to be published in a publicly accessible log.

We will learn how to examine CT logs to discover additional domain names and subdomains for a target organization using two primary resources:

-   <https://censys.io>
-   <https://crt.sh>

Although the website is excellent, we would like to have this information organized and be able to combine it with other sources found throughout the information-gathering process. Let us perform a curl request to the target website asking for a JSON output as this is more manageable for us to process. We can do this via the following commands:

    export TARGET="facebook.com"
    curl -s "https://crt.sh/?q=${TARGET}&output=json" | jq -r '.[] | "\(.name_value)\n\(.common_name)"' | sort -u > "${TARGET}_crt.sh.txt"

    head -n20 facebook.com_crt.sh.txt
    
    *.adtools.facebook.com
    *.ak.facebook.com
    *.ak.fbcdn.net
    *.alpha.facebook.com
    *.assistant.facebook.com
    *.beta.facebook.com
    *.channel.facebook.com
    *.cinyour.facebook.com
    *.cinyourrc.facebook.com
    *.connect.facebook.com
    *.cstools.facebook.com
    *.ctscan.facebook.com
    *.dev.facebook.com
    *.dns.facebook.com
    *.extern.facebook.com
    *.extools.facebook.com
    *.f--facebook.com
    *.facebook.com
    *.facebookcorewwwi.onion
    *.facebookmail.com

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<thead>
<tr>
<th scope="col" class="org-left"><code>curl -s</code></th>
<th scope="col" class="org-left">Issue the request with minimal output.Issue the request with minimal output.</th>
</tr>
</thead>

<tbody>
<tr>
<td class="org-left"><code>https://crt.sh/?q=&lt;DOMAIN&gt;&amp;output=json</code></td>
<td class="org-left">Ask for the json output.</td>
</tr>


<tr>
<td class="org-left"><code>jq -r '.[]' "\(.name_value)\n\(.common_name)"'</code></td>
<td class="org-left">Process the json output and print certificate's name value and common name one per line.</td>
</tr>


<tr>
<td class="org-left"><code>sort -u</code></td>
<td class="org-left">Sort alphabetically the output provided and removes duplicates.</td>
</tr>
</tbody>
</table>

We also can manually perform this operation against a target using OpenSSL via:

    export TARGET="facebook.com"
    export PORT="443"
    openssl s_client -ign_eof 2>/dev/null <<<$'HEAD / HTTP/1.0\r\n\r' -connect "${TARGET}:${PORT}" | openssl x509 -noout -text -in - | grep 'DNS' | sed -e 's|DNS:|\n|g' -e 's|^\*.*||g' | tr -d ',' | sort -u
    
      *.facebook.com
    *.facebook.net
    *.fbcdn.net
    *.fbsbx.com


<a id="org56e9529"></a>

### Automating Passive Subdomain Enumeration

[TheHarvester](https://github.com/laramies/theHarvester) is a simple-to-use yet powerful and effective tool for early-stage penetration testing and red team engagements. We can use it to gather information to help identify a company's attack surface. The tool collects `emails`, `names`, `subdomains`, `IP addresses`, and `URLs` from various public data sources for passive information gathering. For now, we will use the following modules:

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left"><a href="http://www.baidu.com/">Baidu</a></td>
<td class="org-left">Baidu search engine.</td>
</tr>


<tr>
<td class="org-left"><code>Bufferoverun</code></td>
<td class="org-left">Uses data from Rapid7's Project Sonar - www.rapid7.com/research/project-sonar/</td>
</tr>


<tr>
<td class="org-left"><a href="https://crt.sh/">Crtsh</a></td>
<td class="org-left">Comodo Certificate search.</td>
</tr>


<tr>
<td class="org-left"><a href="https://hackertarget.com/">Hackertarget</a></td>
<td class="org-left">Online vulnerability scanners and network intelligence to help organizations.</td>
</tr>


<tr>
<td class="org-left"><code>0tx</code></td>
<td class="org-left">AlienVault Open Threat Exchange - <a href="https://otx.alienvault.com">https://otx.alienvault.com</a></td>
</tr>


<tr>
<td class="org-left"><a href="https://rapiddns.io/">Rapiddns</a></td>
<td class="org-left">DNS query tool, which makes querying subdomains or sites using the same IP easy.</td>
</tr>


<tr>
<td class="org-left"><a href="https://github.com/aboul3la/Sublist3r">Sublist3r</a></td>
<td class="org-left">Fast subdomains enumeration tool for penetration testers</td>
</tr>


<tr>
<td class="org-left"><a href="http://www.threatcrowd.org/">Threacrowd</a></td>
<td class="org-left">Open source threat intelligence.</td>
</tr>


<tr>
<td class="org-left"><a href="https://www.threatminer.org/">Threatminer</a></td>
<td class="org-left">Data mining for threat intelligence.</td>
</tr>


<tr>
<td class="org-left"><code>Trello</code></td>
<td class="org-left">Search Trello boards (Uses Google search)</td>
</tr>


<tr>
<td class="org-left"><a href="https://urlscan.io/">Urlscan</a></td>
<td class="org-left">A sandbox for the web that is a URL and website scanner.</td>
</tr>


<tr>
<td class="org-left"><code>Vhost</code></td>
<td class="org-left">Bing virtual hosts search.</td>
</tr>


<tr>
<td class="org-left"><a href="https://www.virustotal.com/gui/home/search">Virustotal</a></td>
<td class="org-left">Domain search.</td>
</tr>


<tr>
<td class="org-left"><a href="https://www.zoomeye.org/">Zoomeye</a></td>
<td class="org-left">A Chinese version of Shodan.</td>
</tr>
</tbody>
</table>

To automate this, we will create a file called sources.txt with the following contents.

    cat sources.txt
    
    baidu
    bufferoverun
    crtsh
    hackertarget
    otx
    projecdiscovery
    rapiddns
    sublist3r
    threatcrowd
    trello
    urlscan
    vhost
    virustotal
    zoomeye

Once the file is created, we will execute the following commands to gather information from these sources.

    export TARGET="facebook.com"
    cat sources.txt | while read source; do theHarvester -d "${TARGET}" -b $source -f "${source}_${TARGET}";done
    
    <Snip>

When the process finishes, we can extract all the subdomains found and sort them via the following command:

    cat *.json | jq -r '.hosts[]' 2>/dev/null | cut -d':' -f 1 | sort -u > "${TARGET}_theHarvester.txt"

Now we can merge all the passive reconnaissance files via:

    cat facebook.com_*.txt | sort -u > facebook.com_subdomains_passive.txt
    cat facebook.com_subdomains_passive.txt | wc -l
    
    11947


<a id="org32e4a8b"></a>

## Passive Infrastructure Identification

[Netcraft](https://www.netcraft.com/) can offer us information about the servers without even interacting with them, and this is something valuable from a passive information gathering point of view. We can use the service by visiting <https://sitereport.netcraft.com> and entering the target domain.

Some interesting details we can observe from the report are:

<table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">


<colgroup>
<col  class="org-left" />

<col  class="org-left" />
</colgroup>
<tbody>
<tr>
<td class="org-left"><code>Background</code></td>
<td class="org-left">General information about the domain, including the date it was first seen by Netcraft crawlers.</td>
</tr>


<tr>
<td class="org-left"><code>Network</code></td>
<td class="org-left">Information about the netblock owner, hosting company, nameservers, etc.</td>
</tr>


<tr>
<td class="org-left"><code>Hosting history</code></td>
<td class="org-left">Latest IPs used, webserver, and target OS.</td>
</tr>
</tbody>
</table>

<span class="underline">Wayback Machine</span>
The Internet Archive is an American digital library that provides free public access to digitalized materials, including websites, collected automatically via its web crawlers.

We can access several versions of these websites using the Wayback Machine to find old versions that may have interesting comments in the source code or files that should not be there.

We can also use the tool [waybackurls](https://github.com/tomnomnom/waybackurls) to inspect URLs saved by Wayback Machine and look for specific keywords. Provided we have Go set up correctly on our host, we can install the tool as follows:

    go install github.com/tomnomnom/waybackurls@latest

    waybackurls -dates https://facebook.com > waybackurls.txt
    cat waybackurls.txt
    
    2018-05-20T09:46:07Z http://www.facebook.com./
    2018-05-20T10:07:12Z https://www.facebook.com/
    2018-05-20T10:18:51Z http://www.facebook.com/#!/pages/Welcome-Baby/143392015698061?ref=tsrobots.txt
    2018-05-20T10:19:19Z http://www.facebook.com/


<a id="org26482b8"></a>

# ACTIVE Informatin Gathering


<a id="orgc60b326"></a>

## Active Infrastructure Identification


<a id="orgc4704bb"></a>

### HTTP Headers

    curl -I "http://${TARGET}"


<a id="orgf903f12"></a>

### WhatWeb

[Whatweb](https://www.morningstarsecurity.com/research/whatweb) recognizes web technologies, including content management systems (CMS), blogging platforms, statistic/analytics packages, JavaScript libraries, web servers, and embedded devices.

    whatweb -a3 https://www.facebook.com -v

We also would want to install [Wappalyzer](https://www.wappalyzer.com/) as a browser extension. It has similar functionality to Whatweb, but the results are displayed while navigating the target URL.


<a id="org74c1e0e"></a>

### WafW00f

[WafW00f](https://github.com/EnableSecurity/wafw00f) is a web application firewall (`WAF`) fingerprinting tool that sends requests and analyses responses to determine if a security solution is in place.

<span class="underline">Installing</span>

    sudo apt install wafw00f -y

We can use options like `-a` to check all possible WAFs in place instead of stopping scanning at the first match, read targets from an input file via the `-i` flag, or proxy the requests using the `-p` option.

    wafw00f -v https://www.tesla.com


<a id="orgc958118"></a>

### Acquatone

[Aquatone](https://github.com/michenriksen/aquatone) is a tool for automatic and visual inspection of websites across many hosts and is convenient for quickly gaining an overview of HTTP-based attack surfaces by scanning a list of configurable ports, visiting the website with a headless Chrome browser, and taking a screenshot. This is helpful, especially when dealing with huge subdomain lists.

<span class="underline">Installing</span>

    sudo apt install golang chromium-driver
    go get github.com/michenriksen/aquatone
    xport PATH="$PATH":"$HOME/go/bin"

Now, it's time to use cat in our subdomain list and pipe the command to aquatone via:

    cat facebook_aquatone.txt | aquatone -out ./aquatone -screenshot-timeout 1000

When it finishes, we will have a file called `aquatone_report.html` where we can see screenshots, technologies identified, server response headers, and HTML.


<a id="org763f317"></a>

## Active Subdomain Enum

We can perform active subdomain enumeration probing the infrastructure managed by the target organization or the 3rd party DNS servers we have previously identified. In this case, the amount of traffic generated can lead to the detection of our reconnaissance activities.


<a id="org1d05a6e"></a>

### ZoneTrasfer

The zone transfer is how a secondary DNS server receives information from the primary DNS server and updates it.

For example, we will use the <https://hackertarget.com/zone-transfer/> service and the `zonetransfer.me` domain to have an idea of the information that can be obtained via this technique.

A manual approach will be the following set of commands:

1.  Identifying Nameservers
    
        nslookup -type=NS zonetransfer.me
        
        Server:		10.100.0.1
        Address:	10.100.0.1#53
        
        Non-authoritative answer:
        zonetransfer.me	nameserver = nsztm2.digi.ninja.
        zonetransfer.me	nameserver = nsztm1.digi.ninja.
    
    Perform the Zone transfer using `-type=any` and  `-query=AXFR` parameters
2.  Testing for ANY and AXFR Zone Transfer
    
        nslookup -type=any -query=AXFR zonetransfer.me nsztm1.digi.ninja
        
        Server:		nsztm1.digi.ninja
        Address:	81.4.108.41#53
        
        zonetransfer.me
                origin = nsztm1.digi.ninja
                mail addr = robin.digi.ninja
                serial = 2019100801
                refresh = 172800
                retry = 900
        <Snip>


<a id="org802df77"></a>

### Gobuster

Gobuster is a tool that we can use to perform subdomain enumeration. It is especially interesting for us the patterns options as we have learned some naming conventions from the passive information gathering we can use to discover new subdomains following the same pattern.

<span class="underline">GoBuster - patterns.txt</span>

    lert-api-shv-{GOBUSTER}-sin6
    atlas-pp-shv-{GOBUSTER}-sin6

The next step will be to launch `gobuster` using the `dns` module, specifying the following options:

-   `dns`: Launch the DNS module
-   `-q`: Don't print the banner and other noise.
-   `-r`: Use custom DNS server
-   `-d`: A target domain name
-   `-p`: Path to the patterns file
-   `-w`: Path to the wordlist
-   `-o`: Output file

<span class="underline">Gobuster - DNS</span>

    export TARGET="facebook.com"
    export NS="d.ns.facebook.com"
    export WORDLIST="numbers.txt"
    gobuster dns -q -r "${NS}" -d "${TARGET}" -w "${WORDLIST}" -p ./patterns.txt -o "gobuster_${TARGET}.txt"
    
    Found: lert-api-shv-01-sin6.facebook.com
    Found: atlas-pp-shv-01-sin6.facebook.com
    Found: atlas-pp-shv-02-sin6.facebook.com
    Found: atlas-pp-shv-03-sin6.facebook.com
    Found: lert-api-shv-03-sin6.facebook.com
    Found: lert-api-shv-02-sin6.facebook.com
    Found: lert-api-shv-04-sin6.facebook.com
    Found: atlas-pp-shv-04-sin6.facebook.com


<a id="org4a46c12"></a>

## Virtual Hosts

A virtual host (`vHost`) is a feature that allows several websites to be hosted on a single server. This is an excellent solution if you have many websites and don't want to go through the time-consuming (and expensive) process of setting up a new web server for each one. Imagine having to set up a different webserver for a mobile and desktop version of the same page. There are two ways to configure virtual hosts:

-   `IP`-based virtual hosting
-   `Name`-based virtual hosting


<a id="org3cb5308"></a>

### IP-based Virtual Hosting

Allows multiple IP addresses on a single host, enabling different servers to be addressed independently under distinct IP addresses. Each server or virtual server can bind to one or more of these IP addresses, presenting autonomy from a client perspective.


<a id="orgde50425"></a>

### Name-based Virtual Hosting

Differentiates services requested based on domains at the application level. Multiple domain names can share a single IP address, with internal separation accomplished through unique folders or directories. For instance, different domains point to specific folders on a server, allowing distinct content for each domain.

During subdomain discovery, instances of shared IP addresses among different subdomains suggest the possibility of virtual hosts or multiple servers potentially managed through a proxy.

Imagine we have identified a web server at 192.168.10.10 during an internal pentest, and it shows a default website using the following command. Are there any virtual hosts present?

    curl -s http://192.168.10.10
    
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
    <Snip>

    curl -s http://192.168.10.10 -H "Host: randomtarget.com"
    
    <html>
        <head>
            <title>Welcome to randomtarget.com!</title>
        </head>
        <body>
            <h1>Success! The randomtarget.com server block is working!</h1>
        </body>
    </html>  

Now we can automate this by using a dictionary file of possible vhost names

<span class="underline">vHosts List</span>

    app
    blog
    dev-admin
    forum
    help
    m
    my
    shop
    some
    store	
    support
    www

<span class="underline">vHost Fuzzing</span>

    cat ./vhosts | while read vhost;do echo "\n********\nFUZZING: ${vhost}\n********";curl -s -I http://192.168.10.10 -H "HOST: ${vhost}.randomtarget.com" | grep "Content-Length: ";done


<a id="orgc46945f"></a>

### Automating Virtual Hosts Discovery

We can use this manual approach for a small list of virtual hosts, but it will not be feasible if we have an extensive list. Using [ffuf](https://github.com/ffuf/ffuf), we can speed up the process and filter based on parameters present in the response.

We can match or filter responses based on different options. The web server responds with a default and static website every time we issue an invalid virtual host in the `HOST` header. We can use the filter by size `-fs` option to discard the default response as it will always have the same size.

    ffuf -w ./vhosts -u http://192.168.10.10 -H "HOST: FUZZ.randomtarget.com" -fs 612

where:

-   `-w`: Path to our wordlist
-   `-u`: URL we want to fuzz
-   `-H` "HOST: FUZZ.randomtarget.com": This is the HOST Header, and the word FUZZ will be used as the fuzzing point.
-   `-fs` 612: Filter responses with a size of 612, default response size in this case.


<a id="orgd4ab596"></a>

## Crawling

Crawling a website is the systematic or automatic process of exploring a website to list all of the resources encountered along the way.


<a id="org7d47081"></a>

### FFuF

We can use [ffuf](https://github.com/ffuf/ffuf) to discover files and folders that we cannot spot by simply browsing the website. All we need to do is launch `ffuf` with a list of folders names and instruct it to look recursively through them.

    ffuf -recursion -recursion-depth 1 -u http://192.168.10.10/FUZZ -w /opt/useful/SecLists/Discovery/Web-Content/raft-small-directories-lowercase.txt

-   `-recursion`: Activates the recursive scan.
-   `-recursion-depth`: Specifies the maximum depth to scan.
-   `-u`: Our target URL, and FUZZ will be the injection point.
-   `-w`: Path to our wordlist.


<a id="org6a9e611"></a>

### Sensitive Information Disclosure

It is typical for the webserver and the web application to handle the files it needs to function. However, it is common to find backup or unreferenced files that can have important information or credentials.

We will combine some of the folders we have found before, a list of common extensions, and some words extracted from the website to see if we can find something that should not be there. The first step will be to create a file with the following folder names and save it as `folders.txt`.

    wp-admin
    wp-content
    wp-includes

Next, we will extract some keywords from the website using CeWL.

    cewl -m5 --lowercase -w wordlist.txt http://192.168.10.10

-   `-m5`: minimum length of 5 characters
-   `--lowercase`: convert them to lowercase
-   `-w <FILE>`: save them into a

The next step will be to combine everything in ffuf to see if we can find some juicy information. For this, we will use the following parameters in ffuf:

-   `-w`: We separate the wordlists by coma and add an alias to them to inject them as fuzzing points later
-   `-u`: Our target URL with the fuzzing points.

    ffuf -w ./folders.txt:FOLDERS,./wordlist.txt:WORDLIST,./extensions.txt:EXTENSIONS -u http://192.168.10.10/FOLDERS/WORDLISTEXTENSIONS

