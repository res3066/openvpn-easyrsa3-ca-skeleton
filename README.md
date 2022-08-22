
This is RS' quick and dirty OpenVPN pki wrapper.

Use case:  you want to set up an OpenVPN server and use x.509 certs with it.

OpenVPN assumes that you're running your own CA.  Public browser
cert CAs such as Entrust are neither necessary nor desirable.

If you're already running a CA for other stuff at your organization,
don't try to reuse it.

The workflow works like this:

The server has a key/cert pair with the "server" constraint set.
The client has a key/cert pair with the "client" constraint set.
Both are issued by the same CA.  Both "know about" the CA.
The server optionally knows about the CRL.

Client opens connection to server.  If the client can validate the
server's cert via the CA, it allows the connection to continue.

Client presents its client cert

Server attempts to validate the client cert against the CA, taking
into account the CRL if available.  If the server can validate the
client cert, the connection is allowed to continue.

Handshake takes place, ephemeral session keys are negotiated,
routes and nameservers are pushed to the client.

Presto, vpn connection established!  Note only one simultaneous
session per CN!  You need a different cert for your ipad vs your lap
if you want both to connect at the same time.

Of course, merely having keys, certs, and ca.crt isn't enough.  There's
a wrapper around them on the client side called a .ovpn file - this
allows you to easily import the configuration with the hostname, port and 
protocol, etc.

That's where this script comes in.

This script works with EasyRSA 3.x - find it at https://github.com/OpenVPN/easy-rsa

You've cloned this git repo.  You'll probably delete .git, rename it, git init, push it somewhere else.

Now cd into your renamed directory and get EasyRSA (we recommend a release, not just cloning master) and untar it like so:

```wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.1.0/EasyRSA-3.1.0.tgz
tar xvfpz EasyRSA-3.1.0.tgz
```

At this point you want to edit ovpn-vars to make sure you have the
right directory, hostname, etc set.

If you really want passwords on this stuff feel free.  There are scripts to accommodate that.  My experience is that CAs belong on physically secure media with no passwords set.

```
./create-ca-without-password
```

Answer the questions as the spirit moves you.  When you're done, you'll have a `./pki` directory which will contain your keys and certs.  So far all we've got is the CA keypair and by default it is good for 10 years.  This should do.  Check it out:

```
openssl x509 -text -noout < pki/ca.crt 
```

Now we can create server certs for you to put on the openvpn server:

```
./create-server-certs-without-password vpn1.example.org
```

And we can create some client.example.org

```
./create-ovpn-file myname
```









