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

Missing functionalities
=======================

The current version is very simple and does not allow:
* Revoking certificates
* Import existing certificates

However, the structure of the store is straightforward and theses
operation can be done by using openssl directly until I implement them.

Store configuration
===================

The `ssl` script can read its configuration from the file
`$HOME/.sslstorec`. This file only needed if you want to change the
default behavior. The config file is a script that will be sourced
when `ssl` starts. Thus the syntax is standard shell (`bash` actually)
syntax to set variables. The available variables and their default
values are:

* `ssl_store_folder`: the folder where the store is located. Defaults
to `$HOME/.ssl-store`.
* `ca_validity_days`: the validity period, in days, of the generated
  CA certificates. Defaults to `3650`
* `cert_validity_days`: the validity period, in days, of the generated
  certificates. Defaults to `365`

Store structure
===============
`ssl init` creates the folder and, if the `git` command is available,
inits the folder as a git repository. If you create a CA named `MyCA`
and generate a  certificate the store structure will be:

    .
    └── MyCA
        ├── cacert.pem
        ├── ca.cnf
        ├── certindex.txt
        ├── certindex.txt.attr
        ├── certindex.txt.attr.old
        ├── certindex.txt.old
        ├── certs
        │   ├── 1000.pem
        ├── private
        │   ├── cakey.pem
        │   └── www.mydomain.org-key.pem
        ├── serial
        └── serial.old
    

`cacert.pem` and `cakey.pem` are the CA certificate and the CA private
key respectively. The files `1000.pem` and `www.mydomain.org-key.pem`
are again the certificate and the private key for the fqdn
`www.mydomain.org`.

The umask of the process running the script is set to `077` thus only
the user that executes the script will have read and write rights on
the created files.


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
    $ ssl gencert
    Enter the FQDN of the certificate to generate (e.g. host.domain.name)
    www.mydomain.org
    Select the CA to use
    1) MyCA
    #? 1
    Generating a 4096 bit RSA private key
    ..................++
    ..++
    writing new private key to '/home/hauspie/.ssl-store/MyCA/private/www.mydomain.org-key.pem'
    -----
    Using configuration from /tmp/tmp.LaCk6VIHXe
    Enter pass phrase for /home/hauspie/.ssl-store/MyCA/private/cakey.pem:
    Check that the request matches the signature
    Signature ok
    The Subject's Distinguished Name is as follows
    organizationName      :PRINTABLE:'MyCA'
    organizationalUnitName:PRINTABLE:'MyCA'
    localityName          :PRINTABLE:'Seclin'
    stateOrProvinceName   :PRINTABLE:'Nord'
    countryName           :PRINTABLE:'FR'
    commonName            :PRINTABLE:'www.mydomain.org'
    Certificate is to be certified until Feb 18 10:16:08 2016 GMT (365 days)
    
    Write out database with 1 new entries
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number: 4096 (0x1000)
        Signature Algorithm: sha1WithRSAEncryption
            Issuer: O=MyCA, OU=MyCA, L=Seclin, ST=Nord, C=FR/emailAddress=ssl-admin@mydomain.org, CN=MyCA
            Validity
                Not Before: Feb 18 10:16:08 2015 GMT
                Not After : Feb 18 10:16:08 2016 GMT
            Subject: C=FR, ST=Nord, O=MyCA, OU=MyCA, CN=www.mydomain.org
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (4096 bit)
                    Modulus:
                        00:c0:4d:86:33:93:48:de:3b:55:14:5a:a4:50:be:
                        a6:1b:86:f4:47:c8:84:31:b2:64:ff:56:82:2b:42:
                        69:a3:f1:59:c5:91:2a:2b:ae:c7:77:66:fe:01:bb:
                        f1:33:8c:23:b1:d7:cb:5c:78:b5:d3:26:84:4f:2b:
                        36:4f:bd:eb:92:34:e2:1e:18:d7:6d:a1:16:e3:fe:
                        47:1c:b1:1e:f9:20:03:10:47:dc:5f:eb:b4:c5:20:
                        2c:9b:a4:71:bc:23:43:9f:f5:7c:c4:88:bd:f8:58:
                        63:56:e8:00:78:68:62:ff:58:5a:86:94:57:bf:f2:
                        18:c0:ef:37:39:2f:b6:21:5d:9c:f6:7c:5a:70:1b:
                        2a:6d:16:42:23:9e:67:de:d2:fb:72:a1:d6:03:5a:
                        7b:73:17:ee:af:9e:d8:5e:7a:13:63:e3:c0:7a:ff:
                        fe:47:2f:32:c5:ab:b5:7e:85:62:d9:8a:03:fb:64:
                        bf:18:55:61:05:44:4f:51:9d:95:3d:6c:09:88:ee:
                        bf:e3:19:f1:99:f5:c2:4c:ee:2e:2a:02:51:d5:cb:
                        4e:7a:1f:0d:37:53:61:32:d6:40:bf:c7:54:1f:1b:
                        09:14:72:69:cb:36:6c:53:50:7a:4a:c1:c9:82:4b:
                        d9:05:93:de:ee:2d:c8:86:95:7b:9f:48:1d:78:3f:
                        77:e0:bf:60:62:fe:27:53:a7:4f:4a:8d:fb:05:18:
                        51:90:ac:d5:31:d9:1e:90:5c:49:2f:97:bc:bf:70:
                        ca:b8:fe:89:20:ed:23:5b:e6:7c:65:48:4b:cb:f6:
                        5b:02:e0:a7:6d:8a:26:f5:e2:d8:d6:88:e7:d8:3c:
                        46:14:ab:b9:ac:ab:ff:38:39:b9:4f:67:74:a0:dc:
                        d2:d5:3f:0e:1f:b7:d4:0f:34:d6:4b:32:28:99:e3:
                        b7:6c:db:7e:b3:da:06:42:91:44:30:4a:53:78:d1:
                        83:50:ce:f3:8c:8c:0f:de:9b:b5:7e:18:35:d3:9b:
                        1f:72:1a:2c:16:85:68:a9:54:73:bf:e4:b9:58:0f:
                        20:bb:ad:4a:b7:65:bd:fb:fe:e3:44:16:82:7e:49:
                        bd:07:c6:9a:e4:76:a8:b7:26:a8:e8:84:f9:14:3c:
                        f7:fc:5e:c2:8f:59:a0:ef:13:75:02:cf:09:2c:11:
                        2d:98:57:95:12:b2:57:9c:22:d9:d5:3a:17:1a:20:
                        b9:96:a4:5d:6a:7b:6f:53:d0:35:2f:5a:a1:22:f7:
                        5e:38:bb:ad:a3:79:82:b1:0c:14:83:35:9d:d0:a2:
                        0d:fc:f2:34:0d:5b:8b:2b:9f:f8:06:09:88:c8:e1:
                        c8:7b:56:9f:63:02:d1:0a:74:b1:ed:db:e4:65:4b:
                        f9:a6:ab
                    Exponent: 65537 (0x10001)
        Signature Algorithm: sha1WithRSAEncryption
             8e:4c:2c:1b:08:32:11:18:10:f7:e9:95:d1:fb:23:55:7a:57:
             e8:0c:40:7b:0b:ba:3f:43:e5:1d:28:55:45:dc:89:fc:81:2c:
             23:81:08:a1:57:41:04:17:b9:b8:81:3b:38:49:73:c8:3c:45:
             86:fa:a5:c7:c3:cc:91:89:59:7c:18:58:e3:31:de:3f:2f:22:
             88:55:8a:db:7e:2e:53:95:d3:de:5d:65:73:9a:fd:0e:c3:eb:
             a6:57:7e:f4:85:2c:af:bc:75:72:73:53:43:b0:17:3d:4a:4e:
             24:87:b2:bb:37:ba:1b:6a:81:06:7d:77:19:eb:2b:0a:2c:e5:
             f8:83:4e:c8:95:7c:38:2a:b3:e3:10:49:45:43:1e:9d:e1:a5:
             5c:eb:56:70:6c:36:8e:0c:62:ad:b5:11:7c:8d:89:82:32:33:
             0d:52:5f:11:c2:cf:bb:d3:f9:c2:2a:ce:c5:a6:65:3f:f4:81:
             a2:b9:ab:50:74:ed:64:37:2c:9e:00:2a:ab:34:2b:32:ae:0c:
             2b:21:01:45:19:0f:5c:a2:61:6f:2c:e0:37:b6:b7:4c:69:f1:
             52:41:8b:81:e1:14:60:6d:5b:57:93:c7:15:4e:e7:08:f8:3d:
             7e:45:2b:cb:57:53:a0:51:7e:d3:c3:55:0a:89:46:31:ba:da:
             4b:42:81:42:35:47:b5:11:03:f9:b9:f3:f0:74:11:c5:6a:bf:
             cb:12:8b:94:b1:60:c8:75:e6:5a:b5:79:ae:a1:a1:04:11:15:
             b0:1e:47:09:31:43:1c:cb:f0:a5:d0:f4:4f:ef:c6:3e:46:3e:
             dd:f9:51:61:18:14:45:7a:52:54:12:cc:ae:71:54:6d:93:79:
             53:79:c3:16:f7:73:2e:f1:a6:c8:d8:18:41:67:0a:3d:71:99:
             22:7f:62:a8:16:ec:44:c3:be:34:2a:00:fe:1f:88:d8:ac:69:
             ee:c1:0e:5e:6a:57:c2:a0:e1:54:66:ae:1f:d8:94:29:52:34:
             58:a9:d4:c5:2b:0a:cc:f6:97:3c:8c:80:86:18:95:b6:ce:28:
             85:49:17:74:2e:ae:f7:bb:5e:82:28:02:11:f5:ec:9c:f0:3d:
             bf:c4:b1:6f:26:80:82:cd:cf:e9:94:03:84:dd:89:27:1e:53:
             e8:d8:7b:b7:22:d0:a5:16:59:3c:5f:5b:1b:f7:9c:78:a9:f8:
             28:54:ae:01:b0:b9:c8:75:5b:bf:9e:3a:9d:3d:28:b8:34:dc:
             39:ef:1e:ef:bb:15:19:b7:24:59:e3:59:5e:a3:25:db:c2:58:
             32:83:16:8a:50:08:80:8c:4c:fe:da:b7:3f:8a:e3:b8:df:d2:
             6b:f8:b4:be:f1:f3:79:2f
    -----BEGIN CERTIFICATE-----
    MIIFTDCCAzQCAhABMA0GCSqGSIb3DQEBBQUAMIGBMQ0wCwYDVQQKEwRNeUNBMQ0w
    CwYDVQQLEwRNeUNBMQ8wDQYDVQQHEwZTZWNsaW4xDTALBgNVBAgTBE5vcmQxCzAJ
    BgNVBAYTAkZSMSUwIwYJKoZIhvcNAQkBFhZzc2wtYWRtaW5AbXlkb21haW4ub3Jn
    MQ0wCwYDVQQDEwRNeUNBMB4XDTE1MDIxODEwMTYwOFoXDTE2MDIxODEwMTYwOFow
    VTELMAkGA1UEBhMCRlIxDTALBgNVBAgTBE5vcmQxDTALBgNVBAoTBE15Q0ExDTAL
    BgNVBAsTBE15Q0ExGTAXBgNVBAMTEHd3dy5teWRvbWFpbi5vcmcwggIiMA0GCSqG
    SIb3DQEBAQUAA4ICDwAwggIKAoICAQDATYYzk0jeO1UUWqRQvqYbhvRHyIQxsmT/
    VoIrQmmj8VnFkSorrsd3Zv4Bu/EzjCOx18tceLXTJoRPKzZPveuSNOIeGNdtoRbj
    /kccsR75IAMQR9xf67TFICybpHG8I0Of9XzEiL34WGNW6AB4aGL/WFqGlFe/8hjA
    7zc5L7YhXZz2fFpwGyptFkIjnmfe0vtyodYDWntzF+6vntheehNj48B6//5HLzLF
    q7V+hWLZigP7ZL8YVWEFRE9RnZU9bAmI7r/jGfGZ9cJM7i4qAlHVy056Hw03U2Ey
    1kC/x1QfGwkUcmnLNmxTUHpKwcmCS9kFk97uLciGlXufSB14P3fgv2Bi/idTp09K
    jfsFGFGQrNUx2R6QXEkvl7y/cMq4/okg7SNb5nxlSEvL9lsC4Kdtiib14tjWiOfY
    PEYUq7msq/84OblPZ3Sg3NLVPw4ft9QPNNZLMiiZ47ds236z2gZCkUQwSlN40YNQ
    zvOMjA/em7V+GDXTmx9yGiwWhWipVHO/5LlYDyC7rUq3Zb37/uNEFoJ+Sb0Hxprk
    dqi3JqjohPkUPPf8XsKPWaDvE3UCzwksES2YV5USslecItnVOhcaILmWpF1qe29T
    0DUvWqEi9144u62jeYKxDBSDNZ3Qog388jQNW4srn/gGCYjI4ch7Vp9jAtEKdLHt
    2+RlS/mmqwIDAQABMA0GCSqGSIb3DQEBBQUAA4ICAQCOTCwbCDIRGBD36ZXR+yNV
    elfoDEB7C7o/Q+UdKFVF3In8gSwjgQihV0EEF7m4gTs4SXPIPEWG+qXHw8yRiVl8
    GFjjMd4/LyKIVYrbfi5TldPeXWVzmv0Ow+umV370hSyvvHVyc1NDsBc9Sk4kh7K7
    N7obaoEGfXcZ6ysKLOX4g07IlXw4KrPjEElFQx6d4aVc61ZwbDaODGKttRF8jYmC
    MjMNUl8Rws+70/nCKs7FpmU/9IGiuatQdO1kNyyeACqrNCsyrgwrIQFFGQ9comFv
    LOA3trdMafFSQYuB4RRgbVtXk8cVTucI+D1+RSvLV1OgUX7Tw1UKiUYxutpLQoFC
    NUe1EQP5ufPwdBHFar/LEouUsWDIdeZatXmuoaEEERWwHkcJMUMcy/Cl0PRP78Y+
    Rj7d+VFhGBRFelJUEsyucVRtk3lTecMW93Mu8abI2BhBZwo9cZkif2KoFuxEw740
    KgD+H4jYrGnuwQ5ealfCoOFUZq4f2JQpUjRYqdTFKwrM9pc8jICGGJW2ziiFSRd0
    Lq73u16CKAIR9eyc8D2/xLFvJoCCzc/plAOE3YknHlPo2Hu3ItClFlk8X1sb95x4
    qfgoVK4BsLnIdVu/njqdPSi4NNw57x7vuxUZtyRZ41leoyXbwlgygxaKUAiAjEz+
    2rc/iuO439Jr+LS+8fN5Lw==
    -----END CERTIFICATE-----
    Data Base Updated
    [master a014caa] Adds certificate www.mydomain.org to CA MyCA
     4 files changed, 165 insertions(+), 1 deletion(-)
     create mode 100644 MyCA/certs/1000.pem
     create mode 100644 MyCA/private/www.mydomain.org-key.pem
    $ ssl cert www.mydomain.org
    -----BEGIN CERTIFICATE-----
    MIIFTDCCAzQCAhABMA0GCSqGSIb3DQEBBQUAMIGBMQ0wCwYDVQQKEwRNeUNBMQ0w
    CwYDVQQLEwRNeUNBMQ8wDQYDVQQHEwZTZWNsaW4xDTALBgNVBAgTBE5vcmQxCzAJ
    BgNVBAYTAkZSMSUwIwYJKoZIhvcNAQkBFhZzc2wtYWRtaW5AbXlkb21haW4ub3Jn
    MQ0wCwYDVQQDEwRNeUNBMB4XDTE1MDIxODEwMTYwOFoXDTE2MDIxODEwMTYwOFow
    VTELMAkGA1UEBhMCRlIxDTALBgNVBAgTBE5vcmQxDTALBgNVBAoTBE15Q0ExDTAL
    BgNVBAsTBE15Q0ExGTAXBgNVBAMTEHd3dy5teWRvbWFpbi5vcmcwggIiMA0GCSqG
    SIb3DQEBAQUAA4ICDwAwggIKAoICAQDATYYzk0jeO1UUWqRQvqYbhvRHyIQxsmT/
    VoIrQmmj8VnFkSorrsd3Zv4Bu/EzjCOx18tceLXTJoRPKzZPveuSNOIeGNdtoRbj
    /kccsR75IAMQR9xf67TFICybpHG8I0Of9XzEiL34WGNW6AB4aGL/WFqGlFe/8hjA
    7zc5L7YhXZz2fFpwGyptFkIjnmfe0vtyodYDWntzF+6vntheehNj48B6//5HLzLF
    q7V+hWLZigP7ZL8YVWEFRE9RnZU9bAmI7r/jGfGZ9cJM7i4qAlHVy056Hw03U2Ey
    1kC/x1QfGwkUcmnLNmxTUHpKwcmCS9kFk97uLciGlXufSB14P3fgv2Bi/idTp09K
    jfsFGFGQrNUx2R6QXEkvl7y/cMq4/okg7SNb5nxlSEvL9lsC4Kdtiib14tjWiOfY
    PEYUq7msq/84OblPZ3Sg3NLVPw4ft9QPNNZLMiiZ47ds236z2gZCkUQwSlN40YNQ
    zvOMjA/em7V+GDXTmx9yGiwWhWipVHO/5LlYDyC7rUq3Zb37/uNEFoJ+Sb0Hxprk
    dqi3JqjohPkUPPf8XsKPWaDvE3UCzwksES2YV5USslecItnVOhcaILmWpF1qe29T
    0DUvWqEi9144u62jeYKxDBSDNZ3Qog388jQNW4srn/gGCYjI4ch7Vp9jAtEKdLHt
    2+RlS/mmqwIDAQABMA0GCSqGSIb3DQEBBQUAA4ICAQCOTCwbCDIRGBD36ZXR+yNV
    elfoDEB7C7o/Q+UdKFVF3In8gSwjgQihV0EEF7m4gTs4SXPIPEWG+qXHw8yRiVl8
    GFjjMd4/LyKIVYrbfi5TldPeXWVzmv0Ow+umV370hSyvvHVyc1NDsBc9Sk4kh7K7
    N7obaoEGfXcZ6ysKLOX4g07IlXw4KrPjEElFQx6d4aVc61ZwbDaODGKttRF8jYmC
    MjMNUl8Rws+70/nCKs7FpmU/9IGiuatQdO1kNyyeACqrNCsyrgwrIQFFGQ9comFv
    LOA3trdMafFSQYuB4RRgbVtXk8cVTucI+D1+RSvLV1OgUX7Tw1UKiUYxutpLQoFC
    NUe1EQP5ufPwdBHFar/LEouUsWDIdeZatXmuoaEEERWwHkcJMUMcy/Cl0PRP78Y+
    Rj7d+VFhGBRFelJUEsyucVRtk3lTecMW93Mu8abI2BhBZwo9cZkif2KoFuxEw740
    KgD+H4jYrGnuwQ5ealfCoOFUZq4f2JQpUjRYqdTFKwrM9pc8jICGGJW2ziiFSRd0
    Lq73u16CKAIR9eyc8D2/xLFvJoCCzc/plAOE3YknHlPo2Hu3ItClFlk8X1sb95x4
    qfgoVK4BsLnIdVu/njqdPSi4NNw57x7vuxUZtyRZ41leoyXbwlgygxaKUAiAjEz+
    2rc/iuO439Jr+LS+8fN5Lw==
    -----END CERTIFICATE-----
    $ ssl key www.mydomain.org
    -----BEGIN PRIVATE KEY-----
    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
    Z1qClYxHBR9zkXG6Y8/niNIDpujEWA==
    -----END PRIVATE KEY-----


