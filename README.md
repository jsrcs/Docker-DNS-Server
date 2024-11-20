# Objective
Create a "split-horizon" DNS server which resolves private and public IPs with A records using BIND9 deployed on Docker images, on a default ArchLinux installation.
# Steps
1. Install Docker on the machine.
2. Set up the `docker-compose.yml` file that includes the BIND9 service with permissions and exposed ports.
3. Create the BIND9 default configuration with forwarders in `named.conf`.
4. Create a zone file for internal name resolution to different devices on the network.
5. Set your DHCP server to forward the newly configured DNS server.
# Install Docker
`sudo pacman -Syy docker docker-compose`
# Set up the Docker composition file.
[image1]
This tells Docker to and how to run bind9.
We will set the `image` as `ubuntu/bind9`.
Run `docker-compose up` to start Docker with the composition configuration file. We will re-run this every time we want to restart the container.
Set the environment variables for permissions, and a timezone.
Allow the application through the TCP and UDP port 53. As `bind9` is an open DNS resolver, make sure to never expose port 53 to the internet, which will be abused and likely crash the DNS server.
Specify the volumes to bind the directories to the ones in the container.
The container will only restart until stopped or reboot.

# Create the BIND9 default configuration
[image2]
We will create an ACL for internal DNS resolving. This will include all of the IP addresses in the homelab network.
We will forward all outgoing connections to the specified DNS servers.
Finally, we will specify a zone in the configuration.
The type will be master.
The file path specified to the zone file will correlate to the bindings made in the Docker composition file.
# Create the zone file
[image3]
We will define the `$TTL` variable as a duration to keep within the cache.
The `$ORIGIN` variable will be the name of the DNS zone you want to configure. This will allow for brevity when creating A records in this document, as the FQDN specified here will not need to be prepended to each record. You have to append a `.` at the end of this variable.
There are two entries we will always need in a DNS zone. The SOA entry, and the nameserver records.
The SOA entry format will start with `@` representing the current DNS or name. The type should be `IN` for "internet." `SOA` will represent the type of record. The next item is the name for the SOA DNS server, which I chose `ns` for brevity. The final field is the email address of the administrator of the zone.
The fields within the parameters will include the 10 digit serial (which is often the date the file was configured), refresh time, retry time, expiry time, and minimum time to live.
The first record entry, it doesn't need a hostname which is usually specified in the first field. Then, we specify the type `IN` for internet, the record type, and the hostname of the SOA server. This defines the nameserver, butis not defined an IP.
The second record entry starts off with the hostname being "ns", type `IN`, and the record type is `A`, and finally the IP address of the name server. This will define the IP of the nameserver as an A record.
The third entry just adds the hostname of the DNS server as an A record so we can ping it.
# Set up the DHCP server to forward newly configured DNS server.
I will log into the router and designate the DNS server IP in the DHCP configuration.
I am using OpenWRT, so I will make the DNS server's IP static, and add it to the list of forwarders with these commands generated by the GUI:
`uci add_list dhcp.cfg01411c.server='192.168.1.104'
uci add network route # =cfg0ec8b4
uci set network.@route[-1].interface='lan'
uci set network.@route[-1].target='192.168.1.104/24'
uci set network.@route[-1].gateway='192.168.1.1'`
