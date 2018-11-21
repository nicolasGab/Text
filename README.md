A collection of tutorials &amp; more.

# SSL CERTIFICATION
Copied from https://medium.com/@rajanmaharjan/secure-your-mongodb-connections-ssl-tls-92e2addb3c89
Written by Rajan Maharjan | Software Engineer | Jun 15, 2017
## Creating own SSL CA to dump our self-signed certificate
We will be using OpenSSL to create own private certificate authority.

The process for creating your own certificate authority is pretty straight forward:

- Create a private key
- Self-sign
- Install root CA on your various workstations

Once you do that, every device that you manage via HTTPS just needs to have its own certificate created with the following steps:

- Create CSR for client
- Sign CSR with root CA key
- Creating the root certificate is easy and can be done quickly. Once you do these steps, you’ll end up with a root SSL certificate that you’ll install on all of your desktops, and a private key you’ll use to sign the certificates that get installed on your various devices.

The first step is to create the private root key which only takes one step. In the example below, I’m creating a 2048 bit key:

` openssl genrsa -out rootCA.key 2048`

Important note: Keep this private key very private. This is the basis of all trust for your certificates, and if someone gets a hold of it, they can generate certificates that your browser will accept. 

You can also create a key that is password protected by adding -des3:
` openssl genrsa -des3 -out rootCA.key 2048`


The next step is to self-sign this certificate.

` openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem`

This will start an interactive script which will ask you for various bits of information. Fill it out as you see fit.

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:Oregon
Locality Name (eg, city) []:Portland
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Overlords
Organizational Unit Name (eg, section) []:IT
Common Name (eg, YOUR name) []:Data Center Overlords
Email Address []:none@none.com

Once done, this will create an SSL certificate called rootCA.pem, signed by itself, valid for 1024 days, and it will act as our root certificate. The interesting thing about traditional certificate authorities is that root certificate is also self-signed. But before you can start your own certificate authority, remember the trick is getting those certs in every browser in the entire world.

## Create A Certificate (Done Once Per Device)

Every device that you wish to install a trusted certificate will need to go through this process. First, just like with the root CA step, you’ll need to create a private key (different from the root CA).

` openssl genrsa -out mongodb.key 2048`

Once the key is created, you’ll generate the certificate signing request.

` openssl req -new -key mongodb.key -out mongodb.csr`

You’ll be asked various questions (Country, State/Province, etc.). Answer them how you see fit. The important question to answer though is common-name.

- Common Name (eg, YOUR name) []: 10.0.0.1

Whatever you see in the address field in your browser when you go to your device must be what you put under common name, even if it’s an IP address. Yes, even an IP (IPv4 or IPv6) address works under common name. If it doesn’t match, even a properly signed certificate will not validate correctly and you’ll get the “cannot verify authenticity” error. Once that’s done, you’ll sign the CSR, which requires the CA root key.

`openssl x509 -req -in mongodb.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out mongodb.crt -days 500 -sha256`

This creates a signed certificate called device.crt which is valid for 500 days (you can adjust the number of days of course, although it doesn’t make sense to have a certificate that lasts longer than the root certificate).

## Create .pem file

` cat mongodb.key mongodb.crt > mongodb.pem`
