## Name: README.txt
## Project: SSL Proxy
## Author: Christian Starkjohann <cs@obdev.at>
## Copyright: sslproxy is distributed under the terms of the GPL v2.0
## Origin: http://www.obdev.at/Products/
============================================================================


What is sslproxy?
=================
sslproxy is a transparent proxy that can translate between encrypted
and unencrypted data transport on socket connections. It also has
a non-transparent mode for automatic encryption-detection on netbios.
If you don't know what SSL is, please skip to the section
"Basics about Cryptography and SSL".

sslproxy has been developed to have more secure servers available for
the secure mode of Sharity (a CIFS/SMB client for Unix). However, the
program can also be used for a multitude of other security related
applications.


What are the typical applications for sslproxy?
===============================================
sslproxy can be used to make a secure server for HTTP, telnet, POP,
CIFS/SMB etc. without changing the server itself. It's therefore
possible to turn an NT file server into a secure file server, to turn a
telnet daemon into an SSL telnet daemon etc.
The opposite is also possible: sslproxy can turn an ordinary client
into it's SSL variant without changing anything on the client. It's
e.g. possible to make secure telnet connections from Windows NT.


Limitations
===========
You can use sslproxy only for protocols which run over a single
TCP port. This does NOT include the FTP protocol! sslproxy does
NOT implement session caching. This makes it a bad (means slow)
choice for protocols which require frequent connection
establishments such as HTTP because the keys have to be
re-negotiated on each connection. Implementing a session cache
would require a major redesign and an additional session cache
daemon. If you want a secure HTTP server, we can REALLY recommend
apache-ssl.


On what platforms does sslproxy run?
====================================
sslproxy has been developed for Unix, but it can be compiled for
Windows with the CYGWIN32 environment from Cygnus Solutions. There's
also a native Windows port available from http://go.to/kai.engert.


Is sslproxy legal?
==================
The answer to this question depends on a lot of factors. It is as
legal as OpenSSL/SSLeay is, because it builds on top of this
library. For a detailed discussion of the legal problems please
read the SSLeay FAQ:

        http://www.psy.uq.edu.au/~ftp/Crypto/

The short but over-simplified answer: If you compile from the sources
and configure SSLeay appropriately for your legal situation, you can
probably create a legal version. The binary distributed on our web site
is not legal in the USA without obtaining a patent license for the RSA
algorithm.


How to use the proxy:
=====================
We'll assume that you have a basic knowledge about SSL and
OpenSSL/SSLeay. If you don't, please read the introduction written
for SSL-samba, which is appended to this README file.

If you operate a server, you need a certificate. In this case you
must have OpenSSL/SSLeay installed somewhere (probably on a Unix
machine) and create the certificate there. For testing, you may use
the dummy certificate provided with sslproxy. The following examples
build on the dummy certificate for simplicity.

The commandline parameters of sslproxy are:
sslproxy [-L <local address>] [-l <local port>]
        [-R <remote address>] [-r <remote port>] [-s] [-n] [-c <certfile>]
        [-k <keyfile>] [-v <verify file>] [-V <verify dir>] [-C] [-P]
valid options are:
-h                  print short help and exit
-L <local address>  IP address where proxy will bind (default=0.0.0.0)
-l <local port>     port number where proxy will bind
-R <remote address> IP address or hostname the proxy will connect to
-r <remote port>    port number the proxy will connect to
-s                  run as server proxy, not client proxy
-n                  do automatic SSL negotiation for netbios
-p <protocol>       protocol to use, may be: ssl23 (default), ssl2, ssl3, tls1
-c <certfile>       use the given certificate in PEM format
-k <keyfile>        use the given key in PEM format (may be contained in cert)
-v <verify file>    file containing the CA's certificate
-V <verify dir>     directory containing CA certificates in hashed format
-C                  use SSL compatibility mode
-P                  require valid peer certificate


Examples:
=========
1. Using sslproxy for a secure CIFS/SMB server on Windows NT
    Windows NT binds the server to the network interface only. This means
    that you must know the IP-address or hostname of the server host.
    Replace "<local address>" with this name:
    
       sslproxy -l1139 -R<local address> -r139 -s -n -c dummyCert.pem

    The server will respond to SSL requests by SSL aware CIFS clients such
    as Sharity on port 1139.

2. Using sslproxy for a secure CIFS/SMB client on Windows NT
    This is a bit tricky because we must tell the CIFS/SMB client in NT to
    connect to the proxy server. This restricts us to port 139. Start the
    proxy with:
    
       sslproxy -L127.0.0.1 -l139 -R<remote host> -r1139 -n

    <remote host> must be replaced by the host you want to connec to. After
    starting the proxy, you can connect the network drive. Instead of the
    remote host you must give the IP address of localhost:
    
       \\127.0.0.1\SHARE

    This does not work on Windows 95, though, because Windows 95 does not
    allow IP addresses for the hostname. You might get this working by
	using the LMHOSTS file.

3. A CIFS/SMB encryption gateway running on a firewall
	You can allow encrypted and unencrypted CIFS connections to one server
	in your private network with automatic negotiation of encryption. Run
	sslproxy on the firewall machine:

	   sslproxy -l139 -R<real-server-IP> -r139 -s -n -c dummyCert.pem
	
	<real-server-IP> is the IP address of the CIFS server in your local
	network.

4. Using sslproxy to connect to secure web servers with telnet
    If you debug problems with a web server, telnet is a handy tool to do
    the request by hand. If you debug a secure web server, you need an
    SSL enabled telnet. This can be achieved with sslproxy:
    
       sslproxy -L127.0.0.1 -l5000 -r<remote host> -r80 -p ssl2 

    Then connect with telnet to the host "localhost" on port 5000. You can
    do that with
       
       telnet localhost 5000

All these examples do not verify certificates. You need to set up a
list of trusted Certification Authorities and pass it with the parameter
"-v" or "-V" to sslproxy if you want to verify certificates. Verification
is switched on with the "-P" parameter.


How do I compile sslproxy from the sources?
===========================================
0. On Windows, download and install CYGWIN32 from http://www.cygnus.com
   (or get the native Windows version from http://go.to/kai.engert).
1. Get and install OpenSSL or SSLeay. We'll assume that you install it at
   the default location, which is "/usr/local/ssl/". For other locations
   please change all absolute pathes referenced in this README accordingly.
   This version of sslproxy has only been tested with OpenSSL 0.9.4 and
   SSLeay 0.9.0. SSLeay can be compiled with a simple "make" in the top
   level directory on most platforms. It won't be optimized if you do that,
   but it will run. For OpenSSL run the "Configure" script as indicated in
   the OpenSSL documentation.
2. Edit the SSLROOT variable in sslproxy's Makefile if necessary.
3. Compile with "make".
4. If everything works well, make has created "sslproxy" on Unix or
   "sslproxy.exe" on Windows.


How to install sslproxy:
========================
On Unix, simply copy the executable "sslproxy" somewhere into your search
path. On Windows, you must also copy the CYGNUS DLL "cygwinb19.dll" into
the search path. This can be either the same directory as for "sslproxy"
or the Windows system32 directory.


###########################################################################
Basics about Cryptography and SSL (OpenSSL or SSLeay)
(copied mostly from samba doc)
###########################################################################

There are many good introductions to cryptography. I assume that the reader
is familiar with the words "encryption", "digital signature" and RSA. If you
don't know these terms, please read the cryptography FAQ part 6 and 7, which
is posted to the usenet newsgroup sci.crypt. It is also available from

    ftp://rtfm.mit.edu/pub/usenet/news.answers/cryptography-faq
and
    http://www.cis.ohio-state.edu/hypertext/faq/usenet/cryptography-faq

I'll concentrate on the questions specific to SSL and samba here.


What is SSL and OpenSSL/SSLeay?
===============================
SSL (Secure Socket Layer) is a protocol for encrypted and authenticated data
transport. It is used by secure web servers for shopping malls, telebanking
and things like that.

SSLeay is a free implementation of the SSL protocol. It is available from

    ftp://ftp.psy.uq.oz.au/pub/Crypto/SSL/

The current version while these lines are written is 0.9.0. The author of
SSLeay is now an employee of a big cryptography company and is therefore
not allowed to continue the maintainance of SSLeay. However, the copyright
of the original library allows other people to continue the effort. This is
done under the name OpenSSL. You can download OpenSSL from

	http://www.openssl.org/

The current version (while these lines are written) is 0.9.4.

Encryption is plagued by legal problems of all kinds. For a discussion of
these please read the documentation of SSLeay, which is available at

    http://www.psy.uq.edu.au/~ftp/Crypto/

To compile samba with SSL support, you must first compile and install
OpenSSL or SSLeay. Both consist of a library (which can be linked to other
applications like samba) and several utility programs needed for key
generation, certification etc. They install to /usr/local/ssl/ by default.


What is a certificate?
======================
A certificate is issued by an issuer, usually a "Certification Authority"
(CA), who confirms something by issuing the certificate. The subject of this
confirmation depends on the CA's policy. CAs for secure web servers (used for
shopping malls etc.) usually only attest that the given public key belongs to
the given domain name. Company-wide CAs might attest that you are an employee
of the company, that you have permissions to use a server or whatever.


What is an X.509 certificate technically?
=========================================
Technically, the certificate is a block of data signed by the certificate
issuer (the CA). The relevant fields are:
   - unique identifier (name) of the certificate issuer
   - time range during that the certificate is valid
   - unique identifier (name) of the certified subject
   - public key of the certified subject
   - the issuer's signature over all of the above
If this certificate should be verified, the verifier must have a table of the
names and public keys of trusted CAs. For simplicity, these tables are lists
of certificates issued by the respective CAs for themselves (self-signed
certificates).



###########################################################################
Setting up files and directories for OpenSSL/SSLeay
###########################################################################

NOTE: The following documentation has been written for SSLeay. Some names
have changed since the library has been renamed to OpenSSL. The general
cryptography utility has been renamed from 'ssleay' to 'openssl' and the
installation does not install a bunch of symbolic links to this utility
under the names 'genrsa', 'gendsa', etc. any more.
The lines written below should work with OpenSSL if you change every
occurence of 'ssleay' to 'openssl'.

The first thing you should do is to change your PATH environment variable to
include the bin directory of SSLeay. E.g.:

    PATH=$PATH:/usr/local/ssl/bin   

Then you should set up SSLeay's random number generator. The state of this
random number generator is held in the file ".rnd" in your home directory. To
set a reasonable random seed, you need random data. Create a random file with
    
    cat >/tmp/rfile.txt

Then type random keys on your keyboard for about one minute. Then type the
EOF character (^D) to terminate input. You may also use your favorite editor
to create the random file, of course. Now you can create a dummy key to
initialize the random number generator:
    
    ssleay genrsa -rand /tmp/rfile.txt > /dev/null
    rm -f /tmp/rfile.txt

Don't forget to delete the file /tmp/rfile.txt. It's more or less equivalent
to your private key!


How to create a keypair
=======================
This is done with 'genrsa' for RSA keys and 'gendsa' for DSA keys. For an RSA
key with 512 bits which is written to the file "key.pem" type:

    ssleay genrsa -des3 512 > key.pem

You will be asked for a pass phrase to protect this key. If you don't want to
protect your private key with a pass phrase, just omit the parameter "-des3".
If you want a different key size, replace the parameter "512". You really
should use a pass phrase.

If you want to remove the pass phrase from a key use:

    ssleay rsa -in key.pem -out newkey.pem

And to add or change a pass phrase:

    ssleay rsa -des3 -in key.pem -out newkey.pem


How to create a dummy certificate
=================================
If you still have your keypair in the file "key.pem", the command

    ssleay req -new -x509 -key key.pem -out cert.pem

will write a self-signed dummy certificate to the file "cert.pem". This can
be used for testing or if only encryption and no certification is needed.
Please bear in mind that encryption without authentication (certification)
can never be secure. It's open to (at least) "man-in-the-middle" attacks.


How to create a certificate signing request
===========================================
You must not simply send your keypair to the CA for signing because it
contains the private key which _must_ be kept secret. A signing request
consists of your public key and some additional information you want to have
bound to that key by the certificate. If you operate a secure web server,
this additional information will (among other things) contain the URL of
your server in the field "Common Name". The certificate signing request is
created from the keypair with the following command (assuming that the key
pair is still in "key.pem"):

    ssleay req -new -key key.pem -out csr.pem

This command will ask you for the information which must be included in the
certificate and will write the signing request to the file "csr.pem". This
signing request is all the CA needs for signing, at least technically. Most
CAs will demand bureaucratic material and money, too.


How to set up a Certification Authority (CA)
============================================
Being a certification authority requires a database that holds the CA's
keypair, the CA's certificate, a list of all signed certificates and other
information. This database is kept in a directory hierarchy below a
configurable starting point. The starting point must be configured in the
ssleay.conf file. This file is at /usr/local/ssl/lib/ssleay.conf if you have
not changed the default installation path.

The first thing you should do is to edit this file according to your needs.
Let's  assume that you want to hold the CA's database at the directory
"/usr/local/ssl/CA". Change the variable "dir" in section "CA_default" to
this path. You may also want to edit the default settings for some variables,
but the values given should be OK. This path is also contained in the shell
script CA.sh, which should be at "/usr/local/ssl/bin/CA.sh". Change the path
in the shell script:

    CATOP=/usr/local/ssl/CA
    CAKEY=./cakey.pem           # relative to $CATOP/
    CACERT=./cacert.pem         # relative to $CATOP/private/

Then create the directory "/usr/local/ssl/CA" and make it writable for the
user that operates the CA. You should also initialize SSLeay as CA user (set
up the random number generator). Now you should call the shell script CA.sh
to set up the initial database:

    CA.sh -newca

This command will ask you whether you want to use an existing certificate or
create one. Just press enter to create a new key pair and certificate. You
will be asked the usual questions for certificates: the country, state, city,
"Common Name", etc. Enter the appropriate values for the CA. When CA.sh
finishes, it has set up a bunch of directories and files. A CA must publish
it's certificate, which is in the file "/usr/local/ssl/CA/cacert.pem".


How to sign a certificate request
=================================
After setting up the CA stuff, you can start signing certificate requests.
Make sure that the SSLeay utilities know where the configuration file is.
The default is compiled in, if you don't use the default location, add the
parameter "-config <cfg-file>". Make also sure that the configuration file
contains the correct path to the CA database. If all this is set up properly,
you can sign the request in the file "csr.pem" with the command:

    ssleay ca -policy policy_anything -days 365 -infiles csr.pem >cert.pem

The resulting certificate (and additional information) will be in "cert.pem".
If you want the certificate to be valid for a period different from 365 days,
simply change the "-days" parameter.


How to install a new CA certificate
===================================
Whereever a certificate must be checked, the CA's certificate must be
available. Let's take the common case where the client verifies the server's
certificate. The case where the server verfies the client's certificate works
the same way. The client receives the server's certificate, which contains
the "Distinguished Name" of the CA. To verify whether the signature in this
certificate is OK, it must look up the public key of that CA. Therefore each
client must hold a database of CAs, indexed by CA name. This database is best
kept in a directory where each file contains the certificate of one CA and is
named after the hashvalue (checksum) of the CA's name. This section describes
how such a database is managed technically. Whether or not to install (and
thereby trust) a CA is a totally different matter.

The client must know the directory of the CA database. This can be configured.
There may also be a configuration option to set up a CA database file which
contains all CA certs in one file. Let's assume that the CA database is kept
in the directory "/usr/local/ssl/certs". The following example assumes that
the CA's certificate is in the file "cacert.pem" and the CA is known as
"myCA". To install the certificate, do the following:

    cp cacert.pem /usr/local/ssl/cers/myCA.pem
    cd /usr/local/ssl/certs
    ln -s myCA.pem `ssleay x509 -noout -hash < myCA.pem`.0

The last command creates a link from the hashed name to the real file.

From now on all certificates signed by the myCA authority will be accepted by
clients that use the directory "/usr/local/ssl/certs/" as their CA certificate
database.


