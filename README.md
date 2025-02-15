## cypherfunk-seeder

Dogecoin-seeder is a crawler for the Dogecoin network, which exposes a list
of reliable nodes via a built-in DNS server.

Features:
* regularly revisits known nodes to check their availability
* bans nodes after enough failures, or bad behaviour
* accepts nodes down to v0.6.4 to request new IP addresses from,
  currently also retruns them as nodes to connect to.
  This should be changed once network adoption is high enough.
* keeps statistics over (exponential) windows of 2 hours, 8 hours,
  1 day and 1 week, to base decisions on.
* very low memory (a few tens of megabytes) and cpu requirements.
* crawlers run in parallel (by default 96 threads simultaneously).

## COMPILATION

You nned to have the Boost dev libraries installed. It depends on
your distribution, but Debian likes have a package called 'libboost-all-dev'
which should give you all the requirements. When your development environment
is set up you can simple compile with 'make'.
If you compile on a Qemu KVM VPS, you may need to edit the first line
of the Makefile, if you get an error about architecture, from: <br>
``` CXXFLAGS = -O3 -g0 -march=native ```
to:
``` CXXFLAGS = -O3 -g0 -mtune-generic ```

## USAGE

Assuming you want to run a dns seed on dnsseed.example.com, you will
need an authorative NS record in example.com's domain record, pointing
to for example vps.example.com:

``` $ dig -t NS dnsseed.example.com ```

``` ;; ANSWER SECTION ``` <br>
``` dnsseed.example.com.   86400    IN      NS     vps.example.com. ```

On the system vps.example.com, you can now run dnsseed:

``` ./dnsseed -h dnsseed.example.com -n vps.example.com ```

That means your domain needs to be able to have arbitrary NS entries
for subdomains. Alternatively, if you want to use the apex domain as seed address,
you can just point this domains NS entry to the server where the seeder is running.

To help understand the NS settings, the request process works like this:
1. The client sends an A request to his DNS Server for dnsseed.example.com.
2. His DNS server goes up the DNS chain and finds which DNS server knows this address.
3. The authorative server for example.com tells us that vps.example.com is the NS server for dnsseed.example.com.
4. The A request gets send to vps.example.com.
5. cypherfunk-seeder takes this request and returns a set of IPs back to the client.

So in short: Set an NS record on dnssseed.example.com pointing to whereever the 
cypherfunk-seeder is running. Make sure the address you put in there 
(vps.example.com) points to this server. Run cypherfunk-seeder with the above parameters.

If all goes right, a query to dnsseed.example.com should 
report several IP addresses as A records. Note that it might take a few hours
for DNS settings to propagate over the internet.

If you want the DNS server to report SOA records, please provide an
e-mailadres (with the @ part replaced by .) using -m.

## RUNNING AS NON-ROOT

Typically, you'll need root privileges to listen to port 53 (name service).

One solution to this is the use of setcap. Install it with your package management system and then use: <br>
``` $ setcap cap_net_bind_service=+eip /path/to/dnsseed ```. <br> This will allow the program to bind to port 53.

Another solution is using an iptables rule (Linux only) to redirect it to
a non-privileged port: <br>

``` $ iptables -t nat -A PREROUTING -p udp --dport 53 -j REDIRECT --to-port 5353 ```

If properly configured, this will allow you to run dnsseed in userspace, using
the -p 5353 option.
