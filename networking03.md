# Networking 03 — Command Utilities

## 1. nslookup

> Used to query DNS and find the IP address associated with a domain name (or vice versa).

```
nslookup google.com
```
Useful for quickly checking if a domain resolves correctly, or which DNS server is answering the query. Also works in reverse: `nslookup 8.8.8.8` tells you the hostname for an IP.

## 2. ping

> Checks whether a host (server/site) is reachable and alive, and measures round-trip latency, by sending ICMP Echo Request packets and waiting for ICMP Echo Reply packets back.

```
ping google.com
ping -c 4 google.com   # send only 4 packets, then stop
```

### How ICMP works with ping (and why your AWS case behaved that way)

* `ping` sends an **ICMP Echo Request** packet to the target.
* If the target is reachable and *allows* ICMP, it replies with an **ICMP Echo Reply** packet.
* This request/reply pair is what `ping` uses to calculate latency and report "packet loss."

**Your AWS example, explained:**
* AWS Security Groups act as a firewall — by default, inbound ICMP traffic is **not** allowed unless you explicitly add a rule for it.
* So when you `ping` your EC2 instance with no ICMP inbound rule, your Echo Request packets reach AWS's network but get **dropped at the security group level** before they ever reach your instance's network stack — the instance never even sees them, so it can't reply. That's your 100% packet loss.
* Once you add an inbound rule allowing **ICMP IPv4** (Type: All ICMP - IPv4, or specifically "Echo Request"), the security group lets those packets through to the instance. The instance's OS-level ICMP stack replies automatically (this is OS/kernel behavior, not something you configure separately), and now you see successful replies — 0% packet loss.
* Important nuance: this is purely a **security group (firewall) issue**, not a routing or DNS issue — the instance was always "up," it just wasn't allowed to receive/respond to ICMP traffic.

## 3. traceroute

> Shows the entire path (every intermediate hop/router) a packet takes to reach a destination, along with the latency at each hop. Useful for diagnosing *where* along the path a slowdown or failure is happening — not just whether the destination is up.

```
traceroute google.com
traceroute -n google.com     # skip DNS resolution of each hop (faster output)
traceroute -m 15 google.com  # set max number of hops to try (default 30)
traceroute -w 2 google.com   # set wait time (seconds) per probe
```

(On Windows the equivalent command is `tracert`.)

## 4. ping vs traceroute — the core difference

> `ping` just tells you **if** the site/server is up (reachable) and how fast the round trip is.
> `traceroute` tells you the **entire path** — every hop/router the packet passes through to get there — which helps pinpoint *where* in the network a problem exists if something's slow or unreachable.

## 5. dig

> A more detailed, DNS-specific lookup tool than `nslookup` — commonly preferred by network/DevOps engineers for querying DNS records.

```
dig google.com              # A record lookup
dig google.com MX           # mail server records
dig google.com NS           # nameserver records
dig +short google.com       # just the IP, no extra output
dig @8.8.8.8 google.com     # query a specific DNS server directly
```

## 6. wget

> Used to download files from the web/servers directly via the command line (HTTP/HTTPS/FTP).

```
wget https://example.com/file.zip
wget -O newname.zip https://example.com/file.zip   # save with a custom filename
wget -c https://example.com/largefile.iso           # resume a partially downloaded file
wget -r https://example.com/                        # recursive download (e.g. whole site/directory)
```

## 7. curl

> Used to transfer data to/from a server — most commonly for testing/interacting with APIs, checking HTTP responses, or downloading files. Much more flexible than `wget` for API work since it supports all HTTP methods, headers, and request bodies.

```
curl https://api.example.com/data                     # GET request, prints response
curl -I https://example.com                            # headers only (HEAD request)
curl -X POST -d '{"key":"value"}' https://api.example.com/data   # POST with a body
curl -H "Authorization: Bearer TOKEN" https://api.example.com/data  # custom header
curl -o file.json https://api.example.com/data          # save output to a file
curl -s https://api.example.com/data                     # silent mode (no progress bar)
```

## 8. ifconfig

> Shows and configures network interfaces on the machine — IP address, netmask, MAC address, and traffic stats for each interface (e.g. `eth0`).

```
ifconfig            # show all interfaces
ifconfig eth0        # show a specific interface
```
Note: `ifconfig` is considered **deprecated/legacy** on modern Linux distros — the current replacement is the `ip` command:
```
ip addr show
ip a          # shorthand
```

## 9. jq (used with curl)

> `jq` is a command-line JSON processor — used to parse, filter, and pretty-print JSON output. Since most APIs return JSON, `curl` + `jq` is a very common combo for quickly reading API responses without a script or GUI tool.

```
curl -s https://api.example.com/data | jq            # pretty-print the raw JSON response
curl -s https://api.example.com/data | jq '.name'      # extract a specific field
curl -s https://api.example.com/users | jq '.[0]'      # get the first item in a JSON array
curl -s https://api.example.com/users | jq '.[] | .email'  # extract a field from every item in an array
```

**Common real use:** `curl -s https://api.github.com/users/octocat | jq '.name, .public_repos'` — hit an API and instantly get just the fields you care about, cleanly formatted.
