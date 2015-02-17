SSL Store
=========

This is a helper script to ease the management of certificates and CA.
It was inspired by **pass**, the standard unix password manager.

It eases the creation of multiple CA and the process of signing
certificates for several fqdns.

If the git command is available on your system, the command
automatically manage the certificate store as a git repository. It
thus commit any new generated CA and certificates helping you to share
the store with fellow admins if needed (before sharing the store, keep
in mind that the private keys of the CAs and the certs are in the
store!).

Usage
=====
* `ssl init`
  Initialize new ssl storage. If git command is installed,
  it will initialize it as a git repository.
* `ssl [ls] [-v]`
  List managed CA. With -v, also lists certificates for
  each CA.
* `ssl ca [caname]`
  Creates a new CA.
* `ssl gencert [certfqdn [caname]]`
  Creates a new certification for a given fqdn signed by a
  managed CA.
* `ssl cert [certfqdn]`
  Ouputs a certificate to stdout.
* `ssl key [certfqdn]`
  Ouputs the private key of a fqdn certificate.
* `ssl git git-args...`
  Runs the git command in the store folder.

Example
=======
    $ ssl ca MyCA
    What is the name of the service using this CA [MyCA]?
    
    What is your country (2 letters internaltional code)?
    FR
    What is your state or province?
    Nord
    What is your city?
    Seclin
    What is the email adresse of the administrator of this CA?
    ssl-admin@mydomain.org    
    Generating a 4096 bit RSA private key
    .........................................................................................................................................................++
    ..................................................................++
    writing new private key to '/home/hauspie/.ssl-store/MyCA/private/cakey.pem'
    Enter PEM pass phrase:
    Verifying - Enter PEM pass phrase:
    -----
    [master d3661b7] Added CA MyCA
     6 files changed, 157 insertions(+)
     create mode 100644 MyCA/ca.cnf
     create mode 100644 MyCA/cacert.pem
     create mode 100644 MyCA/certindex.txt
     create mode 100644 MyCA/certs/.placeholder
     create mode 100644 MyCA/private/cakey.pem
     create mode 100644 MyCA/serial


