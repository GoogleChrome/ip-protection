# Near-path NAT for IP Privacy

## Objective

As browser vendors make efforts to provide their users with additional privacy, IP addresses continue to make it possible to associate users’ activities across origins that otherwise wouldn’t be able to join user identities. This proposal aims to solve this by allowing groups of users to send their traffic through the same server, thus causing all that traffic to appear to originate from the same pool of IP addresses. This server will thus function like a traditional Network Address Translator (NAT). Optionally it allows for encrypting the destination IP on the hop between the client and the privatizing server to hide it from anyone with visibility on that link. This is one proposed solution to this problem, there are plenty of others.

## Solution - MASQUE

<p>To achieve IP privacy for browser users, the browser will utilize <a href="https://datatracker.ietf.org/wg/masque/about/">Multiplexed Application Substrate over QUIC Encryption (MASQUE)</a> to forward its HTTP traffic through an IP privatizing server (IPPS).  The HTTP traffic that servers see will have a source IP address of the IPPS rather than that of the browser, thus hiding the users’ IP address from the server.  The browser will utilize end-to-end encryption via TLS to the target server so the IPPS won’t be privy to the contents of the HTTP traffic.  For HTTP/1 and HTTP/2 traffic the browser’s TLS byte-stream is sent via MASQUE to the IPPS and then the IPPS initiates a TCP connection to the target server carrying that TLS byte-stream, essentially just adding the IP and TCP headers.  For HTTP/3 traffic the browser assembled UDP packets are sent via MASQUE to the IPPS and then the IPPS sends the UDP packets to the target server, essentially just adding the IP header.  The IPPS forwards data received from the target server back to the browser via the same MASQUE streams that the outgoing data traversed.
</p>
<p>The CONNECT and CONNECT-UDP HTTP methods can be used by the browser to inform the IPPS of the destination IP address and port of the target server.  By sharing only the destination IP address with the IPPS and not the target server hostname the privacy properties are improved and the client’s DNS configuration can continue to handle all host resolution.  By comparison a DNS server has more insight into the sites a user browses to because it knows hostnames which contain more information than the IP addresses of the target servers.
</p>
<p>IPPS operators could be subjected to an audit of their privacy practices, e.g. logs retention and logs access controls.  Browser users could choose their IPPS from a list of IPPSs that passed some level of privacy audit.
</p>

## Operating an IP Privatizer Service

<p>Ideally an IP privatizer service like this would be operated as close to on-path as possible, so as to minimize the performance impact of increasing the round trip time. ISPs are in a strong position to operate such a service and in fact some ISPs NAT the majority of their traffic for other reasons (and this service would be roughly indistinguishable from that topology from a web server’s point of view).
</p>
<p>The next best option are large CDNs, which by their nature have a large deployment of edge cache servers as close as possible to users. This service is designed to be as low overhead as possible, primarily forwarding packets between the browser and target servers, so as to not interfere with their intended business purpose.
</p>
<p>An operator of an IP address privatization service should be aware of a few implications of their IP address assignment scheme. First, IP-based geolocation is used by a wide swath of services to serve content that is relevant to users (readers of news in Boston probably don’t care much about restaurant openings in Baltimore) as well as conforming to local laws. An edge caching network running this sort of service should lend itself to a decent mapping of servers to geographic locations, but if for instance a given server is providing service for users in two distinct geographic regions, operators may want to use two IP addresses such that the IPs map well to those regions.
</p>
<p>The other matter of IP and port assignment schemes is to prevent such a service from being abused by fraudsters. Service operators may want to gate access to their service on some kind of registration as a hurdle for fraudsters to overcome. A service operator could look for multiple connections to typically top level domains from a given client as an indicator of fraud and abuse.
</p>
<p>Additionally, the service could allow the browser to signal which connections are to first parties and thus should have stable IP and port mappings between clients and hosts versus connections to third parties which should not be stable across their first parties. The privatizer could then use a seperate pool of well known IPs for first and third parties such that domains that only expect to be connected to as a first party can reject the non-stable set of third parties’ IP addresses.
</p>
<p>Alternatively, the IP privatizer could operate with a range of IPv6 addresses, assigning one IPv6 address per client/first-party-host pair and using the same IPv6 address for connections to third party content embedded in that first party page.  Having stable IP mappings between clients and hosts facilitates pre-existing server-side anti-abuse practices (e.g. detecting and preventing denial-of-service attacks by client IP address).  The IP privatizer could additionally use port assignment within that IP (even and odd for example) to signal to the destination that they are being connected to as a first or third party from the privatizer’s point of view. Hosts which do not expect to be connected to as a third party can drop incoming packets that are designated as third parties.
</p>
<p>To keep stable IP and/or port mappings between clients and first party pages, the clients need to inform the IPPS about which proxied connections are first party and which are third party, and for the third party requests the client needs to inform the IPPS what the corresponding first party is.  This additional information needs to accompany the CONNECT header, as an additional header or some other augmentation mechanism.
</p>

## Alternate Encapsulation Protocols Considered

<p>There are other protocols that can be used between the browser and IPPS.  Each has their particular advantages and disadvantages.  This proposal uses MASQUE as it was designed for IP proxying and leverages the latest transport protocol, HTTP/3.  Some alternatives considered:
</p>
<p><b>General UDP Encapsulation (GUE)</b><br>
GUE is an efficient protocol using UDP to encapsulate packets of different IP protocols.  The simpler nature of the encapsulation could allow for offloading of encapsulation to network hardware, but hardware support is likely many years away.  Using GUE would require browsers to do TCP which would add significant additional complexity.<br>
</p>
<p><b>HTTP(S) CONNECT proxy</b><br>
HTTP CONNECT proxying is another alternative but is unfortunately limited to only proxying TCP presently which would preclude browsers using modern protocols like HTTP/3.<br>
</p>
<p><b>SOCKS proxy</b><br>
SOCKS proxying is another alternative but doesn’t offer encryption of signaling.<br>
</p>
<p><b>VPN</b><br>
The IPPS could operate a VPN.  VPNs typically perform IP packet encapsulation which would either require the browser to run with administrator privileges to encapsulate IP packets from the OS or would require the browser to do TCP which would add significant additional complexity.<br>
</p>
<p><b>SSH Tunnel</b><br>
The IPPS could permit SSH tunnelling but SSH tunnelling is limited to using TCP.
</p>
