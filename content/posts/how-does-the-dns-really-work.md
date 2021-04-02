---
title: "How does the DNS really work"
date: 2021-04-02
description: A long time ago, researchers from the United States have invented a way to send messages across the world without using papers or pigeons. Nowadays, people know about it as the Internet. With the advent of TCP/IP protocol and submarine cable, a person from Vietnam can effectively visit a website hosted in the US, If only that person can remember the website IP address.
---

A long time ago, researchers from the United States have invented a way to send messages across the world without using papers or pigeons. Nowadays, people know about it as _the Internet_. With the advent of TCP/IP protocol and submarine cable, a person from Vietnam can effectively visit a website hosted in the US. If only that person can remember the website IP address which is a combination of four numbers (e.g. _142.250.204.142_) or in 2050, it would be a combination of eight hexadecimal numbers (e.g. _2001:0db8:0000:0000:0000:8a2e:0370:7334_). Why was the Internet designed this way? Why wouldn’t website address be something easy to remember like _221B Baker Street_? The reason is that it’s easier for computers to work with numbers than with arbitrary words, but it’s difficult for mortal humans. And, that gives birth to the Domain Name System (DNS).

DNS is a network of globally distributed and hierarchical **name servers** that can answer queries such as “which IP does [longis.me](http://longis.me) point to?”. **Name server** is simply a server that listens on port 53 and conforms to the DNS protocol. You can even run your own with something like [CoreDNS](https://coredns.io/).

When your browser wants to find out the IP of _google.com_, for example, it doesn’t just query a single centralized name server but multiple of those. This helps make the DNS infrastructure fast and flexible. Let’s get into more details here.

{{< figure src="/img/dns-lookup.png" alt="dns lookup" position="center" caption="Source: [Cloudflare](https://www.cloudflare.com/learning/dns/what-is-dns/)." >}}

1. **Recursive name server:** Your entry point to DNS. Recursive name server will try to answer your queries by going top-down the hierarchy, starting from the _root name server_. Recursive name server can be one run by your ISP (the default case) or a third party like Cloudflare’s 1.1.1.1 or Google’s 8.8.8.8. Normally, recursive name server doesn’t store any domain’s IP and only finds it when asked. Occasionally, it even blocks some domains, especially the ISP’s one…

2. **Root name server:** The mother of all domains. It is maintained by [IANA](https://en.wikipedia.org/wiki/Internet_Assigned_Numbers_Authority) which belongs to the US until 2016 when it is [handed over to the world](https://www.businessinsider.com/the-us-government-no-longer-controls-the-internet-2016-10). Root name server stores the addresses of _TLD name servers_.

3. **TLD name server:** TLD stands for top-level domain, like .com, .net or .org. These name servers are maintained by [non-profit or for-profit organizations](https://en.wikipedia.org/wiki/List_of_Internet_top-level_domains). For example, .com is maintained by [Verisign](https://en.wikipedia.org/wiki/Verisign) and .gay is maintained by [Top Level Design](https://toplevel.design/). TLD name servers stores the addresses of _authoritative name servers_.

4. **Authoritative name server:** The final server that can answer a domain name lookup: _google.com_ points to _142.250.204.142_. Actually, there are many pieces of data a name server can store for one domain. They are called _records_. For example, A record is for IP address, CNAME record is for aliasing to another domain, MX record for emailing, TXT for metadata, etc.

Whew, that’s a lot! When your browser knows which IP a domain points to, it will send a TCP/IP request to it. I’ll write another post about IP routing in the future, so stay tuned!

In reality, because domain name records rarely change, there will be tons of caching alongside the chain. We’ll look into this later though.

## Where do domain registrars fit into the picture?

Registrars are where you purchase domains, like GoDaddy or NameCheap. Whenever you buy a new domain, registrar will sign a lease on your behalf with TLD companies to transfer the domain ownership to yours and decide which authoritative name server to use for that domain. Most registrars will also run their own authoritative name servers for domains bought with their service, though you can later change it to the others like Cloudflare or your own self-hosted name server.

In other words, TLD companies are like wholesalers and registrars are like resellers. The product here is domain ownership.

## There are two hard things

A famous person once said:

> _There are only two hard things in Computer Science: cache invalidation and naming things._

Because DNS is used in almost every request and rarely getting changed, it is a prime candidate for caching. Truly, DNS caching is everywhere, from the browser, operating system to home router, ISP router to the recursive name server itself.

Caching is good for end users but sometimes can be tricky for the agile developers who need to iterate quickly. _nslookup_ and _dig_ are two essential tools for DNS debugging. Here are some other personal tips:

- Lower record TTL (time-to-live), which is how long should records be cached.
- Query authoritative name server directly.
- For Cloudflare: [https://1.1.1.1/purge-cache/](https://1.1.1.1/purge-cache/).

This post barely touches on the subject of using DNS for service abstraction or load balancing, which deserves their own articles. No matter if you are a back-end or a front-end developer, knowing how DNS works could greatly improve your day-to-day productivity.
