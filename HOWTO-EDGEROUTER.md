
Configuration notes on how to set things up on EdgeOS 1.x (Ubiquiti)

Poke a hole in your external firewall to allow OpenVPN traffic through (udp/1194)
```
set firewall name InternetInbound rule 80 action accept
set firewall name InternetInbound rule 80 description 'openvpn remote laptops'
set firewall name InternetInbound rule 80 destination port 1194
set firewall name InternetInbound rule 80 protocol udp
```

Copy in OpenVPN's certs scp them in without a pathname and they'll
end up in a user's home directory; you can then sudo bash from the cli
and move thme to to where they need to live.

Add the prefix that you're using for the openvpn pool to your outbound NAT, especially if you're doing full vpn or pointing at an external nameserver.

In the examples below, 192.0.2.0/24 is your LAN network, 203.0.113.0/24 is the VPN pool.

```
set service nat rule 5002 description 'out 3'
set service nat rule 5002 log disable
set service nat rule 5002 outbound-interface eth0
set service nat rule 5002 protocol all
set service nat rule 5002 source address 203.0.113.0/24
set service nat rule 5002 type masquerade
```

Now add the OpenVPN config.  Note this is for split tunnel VPN.  You could also push route for 0.0.0.0/0 to do "full tunnel VPN"

```
set interfaces openvpn vtun0 description 'laptop vpn'
set interfaces openvpn vtun0 mode server
set interfaces openvpn vtun0 server name-server 8.8.8.8
set interfaces openvpn vtun0 server push-route 192.0.2.0/24
set interfaces openvpn vtun0 server subnet 203.0.113.0/24
set interfaces openvpn vtun0 tls ca-cert-file /config/auth/ssl/ovpn0-cacert.pem
set interfaces openvpn vtun0 tls cert-file /config/auth/ssl/ovpn0-server.pem
set interfaces openvpn vtun0 tls dh-file /config/auth/ssl/ovpn0-dh.pem
set interfaces openvpn vtun0 tls key-file /config/auth/ssl/ovpn0-server.key
```

Beware that certificates have dates and times for valid.  If you aren't using NTP, you should.  Keep your clocks straight to avoid frustration.


