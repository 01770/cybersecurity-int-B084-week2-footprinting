# Week 2: Footprinting and Reconnaissance with Kali Tools

This week's lab was to footprint the website networkwalks.com using six tools that come with Kali Linux: whois, whatweb, nslookup, curl, wafw00f and dnsrecon. I did everything on 10 November 2026 from Kali Linux running in VirtualBox (NAT network). I only used the target given in the lab and only light, information-gathering commands. I did not try to attack or log in to anything.

The text output of every command is in the `outputs` folder and the screenshots are in `screenshots`.

## Quick summary

| Task | Tool | Main thing I found |
|------|------|--------------------|
| 1 | whois | Registered with GoDaddy, owner hidden by a privacy service, name servers at HostGator |
| 2 | whatweb / curl | Apache server running WordPress |
| 3 | nslookup | The domain points to 192.232.216.135 |
| 4 | curl -I | HTTP/2 200, Apache, WordPress cache header, REST API link |
| 5 | wafw00f | The site is behind a ModSecurity WAF |
| 6 | dnsrecon | DNS records, including 8 cPanel SRV records |

## Task 1: whois

Command: `whois networkwalks.com | tee whois.txt`

The domain is registered through GoDaddy, but the owner's contact details are replaced by Domains By Proxy, so I could not see a real name or email. The name servers are NS6135.HOSTGATOR.COM, NS6136.HOSTGATOR.COM, NS29.DOMAINCONTROL.COM and NS30.DOMAINCONTROL.COM. That tells me the site is most likely hosted with HostGator. DNSSEC is unsigned.

Creation date: [FILL IN]. Expiry date: [FILL IN].

Why it matters: an attacker learns the hosting provider and can use the dates and contacts for planning. A defender can keep domain privacy turned on and enable DNSSEC.

Screenshot: `screenshots/1-whois.png`

## Task 2: whatweb

Command: `whatweb networkwalks.com | tee whatweb.txt`

This one did not work for me. whatweb froze at the start every time I ran it. I tried a plain run, running it with `timeout`, setting a user-agent, and turning off IPv6. Nothing changed. ping and curl worked fine from the same Kali machine, so the internet connection was not the problem. My guess is that it is an issue between whatweb (Ruby) and my VirtualBox setup, or that the WAF from Task 5 does not like whatweb's requests, but I could not confirm either.

To still get the information, I read it from the HTTP headers with curl:

`curl -sI https://networkwalks.com | tee whatweb_alt.txt`

From this I could see the server is Apache, the site runs WordPress (the `x-nginx-cache: WordPress` header), and it sets a `__wpdm_client` cookie that belongs to the WordPress Download Manager plugin.

Why it matters: once an attacker knows the software, they can search for known problems in it. A defender should keep WordPress and its plugins updated and hide version details.

Screenshot: `screenshots/2-whatweb-curl.png`

## Task 3: nslookup

Command: `nslookup networkwalks.com | tee nslookup.txt`

The domain resolves to **192.232.216.135**. The answer was non-authoritative, which just means it came from my network's DNS server and not directly from the domain's own name servers.

Why it matters: with the IP address an attacker can target the server directly or look for other sites on the same server. A defender should know what is exposed on that IP.

Screenshot: `screenshots/3-nslookup.png`

## Task 4: HTTP headers

Command: `curl -I https://networkwalks.com | tee curl_headers.txt`

The server answered with HTTP/2 200. The interesting headers were:
- `server: Apache`
- `x-nginx-cache: WordPress`
- a `link` header that points to the WordPress REST API at `/wp-json/`
- a `set-cookie` header with the `__wpdm_client` cookie (marked `secure` and `HttpOnly`)
- `permissions-policy` and `referrer-policy`, which are security-related headers

Why it matters: the headers give away the technology and the API address without even opening the page. A defender can remove headers that are not needed and limit access to the REST API.

Screenshot: `screenshots/4-curl.png`

## Task 5: wafw00f

Command: `wafw00f networkwalks.com | tee wafw00f.txt`

wafw00f reported that the site is behind a **ModSecurity (SpiderLabs) WAF**. It needed 2 requests to find this.

Why it matters: if there is a WAF, simple attacks will probably be blocked or logged, so an attacker has to change their approach. For the owner it is a useful extra layer of protection, as long as its rules are kept up to date.

Screenshot: `screenshots/5-wafw00f.png`

## Task 6: dnsrecon

Command: `dnsrecon -d networkwalks.com | tee dnsrecon.txt`

dnsrecon found 8 SRV records. They are `_autodiscover._tcp.networkwalks.com` records that point to `cpanelemaildiscovery.cpanel.net` on port 443, which shows the hosting uses cPanel. Other records: [FILL IN the SOA, NS, MX and TXT/SPF records from dnsrecon.txt].

Why it matters: DNS records show how email and hosting are set up. A defender should only publish the records that are needed and use a strict SPF policy.

Screenshot: `screenshots/6-dnsrecon.png`

## What I learned

Everything I found came from public information, and I never touched the site in a harmful way. Each tool gives a different piece of the picture, and together they build a good profile of the target. I also learned that tools can fail (whatweb did for me), so it helps to know another way of getting the same information, like reading the headers with curl.
