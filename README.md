# README

Customers normally have their own certificate, which is why to test or reproduce similar enviroments we must generate ssl certificate. The purpose of this role is to automate the process to create a SSL self-signed certificate.

![SSL certificate self-signed](images/ssl_certificate_selfsigned.png)

In the diagram we can see that the last element of the chain, called the root CA, the first element will be the end-entity and any element in between will be an intermediate CA. Also, it can be apreciated the flow that should be followed to sign the certs.

## Role

The role has been done with tags to be executed according to your needs.

```bash
$ ansible-playbook tests/test.yml --list-tags
playbook: tests/test.yml

  play #1 (127.0.0.1): 127.0.0.1	TAGS: []
      TASK TAGS: [create-directories, create-end-entity-cert, create-intermediate-ca, create-root-ca]
```

### variables to take into account.

```bash
      ca_dir: Directory where the certificates will be generated.
      ca_root_common_name: RootCA CN
      ca_intermediate_common_name: Intermediate CA CN
      ## CA Subject Variables
      ca_subject_country: "ES"
      ca_subject_state_province: "Catalunya"
      ca_locality_name: "Barcelona"
      ca_organization_name: "Red Hat Consulting"
      ca_organizational_unit_name: "Red Hat Consulting Certificate Authority"
      endentity_list:
        certificate_file_name: It should have an identifier name without any special character or whitespace.  
          endentity_common_name: end-entity CN
          dns_server:
            - DNS list that certificate should have
```

## Playbook. 
### Playbook example.

It could be found at role/ca-generator/tests/test.yml

###  Plabook execution.

**create-directories**: It will create all the directory structure that will be used.

```bash
$ ansible-playbook tests/test.yml  --tags create-directories
```

The directory structure will be created like below:

```bash
$ tree /tmp/ca/
/tmp/ca/
├── endEntity
│   ├── certs
│   ├── csr
│   └── private
├── intermediateCA
│   ├── certs
│   ├── csr
│   └── private
└── rootCA
    ├── certs
    ├── csr
    └── private
```

**create-root-ca**: It will create the rootCA private key, the rootCA signing request (CSR) and the rootCA cert self-signed.

```bash
$ ansible-playbook tests/test.ym  --tags create-directories,create-root-ca

$ tree /tmp/ca/
/tmp/ca/
├── endEntity
│   ├── certs
│   ├── csr
│   └── private
├── intermediateCA
│   ├── certs
│   ├── csr
│   └── private
└── rootCA
    ├── certs
    │   └── ca-root-cert-self-signed.pem
    ├── csr
    │   └── ca-root.csr
    └── private
        └── ca-root.key

$ openssl rsa -in /tmp/ca/rootCA/private/ca-root.key 
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIJKAIBAAKCAgEAtuJ4+nEctrE9sybTWiIb5zZFbkdyFucTZXW0Pu5Ho5byAVDp
Ry1DOyNPsxQxJByQBcwrrotBUyzto4xw9lNFghGkomVV+kZiNZGQmBPpXuSAeBdx
...
ti6d6Nu8JqlWT2PtbkIO2NOGykB9Zs33EDoTbfaTpI1HAPYzO/coPQDvWbS38rPT
9MLw6ibcT0w8JMg0VDQxVxz+TUNh8Sn48F3E/8NxbeIAeIiDgv3ubyW5c8k=
-----END RSA PRIVATE KEY-----


$ openssl req -text -noout -verify -in /tmp/ca/rootCA/csr/ca-root.csr 
verify OK
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b6:e2:78:fa:71:1c:b6:b1:3d:b3:26:d3:5a:22:
                    ...  
                    64:67:db:29:48:7d:a6:78:f9:b9:b5:76:69:66:4f:
                    35:84:95
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
    Signature Algorithm: sha256WithRSAEncryption
         b5:eb:1b:56:21:7e:46:3f:18:a4:56:df:5a:c2:f0:57:d2:25:
         ...
         c7:98:57:2a:77:53:2d:f9

$ openssl x509 -text -noout -in /tmp/ca/rootCA/certs/ca-root-cert-self-signed.pem 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            51:fc:51:5f:12:27:a3:d2:76:0f:19:f0:6d:ea:87:18:11:85:38:94
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Root CA
        Validity
            Not Before: Mar 23 10:44:56 2021 GMT
            Not After : Mar 21 10:44:56 2031 GMT
        Subject: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:b6:e2:78:fa:71:1c:b6:b1:3d:b3:26:d3:5a:22:
                    ...
                    64:67:db:29:48:7d:a6:78:f9:b9:b5:76:69:66:4f:
                    35:84:95
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE
            X509v3 Subject Key Identifier: 
                D9:AE:54:D3:A1:51:C1:7C:46:34:4C:AE:56:E0:B4:AC:E4:B7:A2:7E
    Signature Algorithm: sha256WithRSAEncryption
         19:01:92:de:42:a0:4a:95:11:ab:09:6f:31:d5:18:e3:72:8d:
         ...
         e3:e5:9c:ab:41:f8:2c:45
```


**create-intermediate-ca**: It will create the intermediateCA private key, the intermediateCA signing request (CSR) and the intermediateCA cert signed by the rootCA. Also a ca-chain certficate will be generated by merging the rootCA and the intermediateCA. The complete chain must be merged in one file if we want to verify the end-entity signed by the intermediateCA.

```bash
$ ansible-playbook tests/test.yml  --tags create-directories,create-root-ca,create-intermediate-ca


$ tree /tmp/ca/
/tmp/ca/
├── endEntity
│   ├── certs
│   ├── csr
│   └── private
├── intermediateCA
│   ├── certs
│   │   ├── ca-chain.pem
│   │   └── ca-intermediate.pem
│   ├── csr
│   │   └── ca-intermediate.csr
│   └── private
│       └── ca-intermediate.key
└── rootCA
    ├── certs
    │   └── ca-root-cert-self-signed.pem
    ├── csr
    │   └── ca-root.csr
    └── private
        └── ca-root.key

$ openssl rsa -in /tmp/ca/intermediateCA/private/ca-intermediate.key
writing RSA key
-----BEGIN RSA PRIVATE KEY-----
MIIJKgIBAAKCAgEA6g1BfdTdItOp0hp+ZoFSSI2roayNN8+/i1kL4mRK9f8kzTRT
...
4SO9ZJqKgYw7RekWSRLJNoSjQPvF+AGRQet5ij1nUCbFx6UQVLg4EUlLcCW8Qw==
-----END RSA PRIVATE KEY-----

$ openssl req -text -noout -verify -in /tmp/ca/intermediateCA/csr/ca-intermediate.csr 
verify OK
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Intermediate CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:ea:0d:41:7d:d4:dd:22:d3:a9:d2:1a:7e:66:81:
                    ...
                    33:9c:e8:c6:f6:95:08:f7:ec:46:ac:32:2b:bd:1f:
                    c4:c8:7f
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
    Signature Algorithm: sha256WithRSAEncryption
         e3:d9:bd:66:62:6b:80:a0:c8:4c:58:b2:87:33:e2:ca:61:22:
         ...
         82:f0:30:ea:2d:58:cd:03

$ openssl x509 -text -noout -in /tmp/ca/intermediateCA/certs/ca-intermediate.pem
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            7a:e3:13:dd:65:b7:74:fb:c1:27:07:c2:a6:5a:2b:1a:1f:eb:8a:90
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Root CA
        Validity
            Not Before: Mar 23 10:51:07 2021 GMT
            Not After : Mar 21 10:51:07 2031 GMT
        Subject: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Intermediate CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (4096 bit)
                Modulus:
                    00:ea:0d:41:7d:d4:dd:22:d3:a9:d2:1a:7e:66:81:
                    ...
                    33:9c:e8:c6:f6:95:08:f7:ec:46:ac:32:2b:bd:1f:
                    c4:c8:7f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
            X509v3 Subject Key Identifier: 
                51:C6:0A:28:F6:7F:C0:45:F5:B0:3E:29:88:59:15:FC:31:4D:2F:33
            X509v3 Authority Key Identifier: 
                keyid:D9:AE:54:D3:A1:51:C1:7C:46:34:4C:AE:56:E0:B4:AC:E4:B7:A2:7E

    Signature Algorithm: sha256WithRSAEncryption
         95:6b:90:b3:21:48:01:cc:e5:e8:1e:33:d4:bc:1f:cc:7c:a6:
         ...
         e2:cc:8c:e1:a0:c0:8b:eb
```
**create-end-entity-cert**: It will create the end-entity private key, the end-entity signing request (CSR) and the end-entity cert signed by the intermediateCA. As mentioned before, the complete chain must be merged in one file if we want to verify the end-entity signed by the intermediateCA.

```bash
$ ansible-playbook tests/test.yml  --tags create-directories,create-root-ca,create-intermediate-ca,create-end-entity-cert,

$ tree /tmp/ca/
/tmp/ca/
├── endEntity
│   ├── certs
│   │   ├── ocp-external.pem
│   │   ├── satcapsule1.pem
│   │   ├── satcapsule2.pem
│   │   ├── satellite.pem
│   │   ├── tower.pem
│   │   ├── wilcard_apps.pem
│   │   └── wilcard_devs.pem
│   ├── csr
│   │   ├── ocp-external.csr
│   │   ├── satcapsule1.csr
│   │   ├── satcapsule2.csr
│   │   ├── satellite.csr
│   │   ├── tower.csr
│   │   ├── wilcard_apps.csr
│   │   └── wilcard_devs.csr
│   └── private
│       ├── ocp-external.key
│       ├── satcapsule1.key
│       ├── satcapsule2.key
│       ├── satellite.key
│       ├── tower.key
│       ├── wilcard_apps.key
│       └── wilcard_devs.key
├── intermediateCA
│   ├── certs
│   │   ├── ca-chain.pem
│   │   └── ca-intermediate.pem
│   ├── csr
│   │   └── ca-intermediate.csr
│   └── private
│       └── ca-intermediate.key
└── rootCA
    ├── certs
    │   └── ca-root-cert-self-signed.pem
    ├── csr
    │   └── ca-root.csr
    └── private
        └── ca-root.key

$ openssl x509 -text -noout -in /tmp/ca/endEntity/certs/tower.pem 
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            0c:42:9f:78:af:a2:d9:1c:5e:c0:ca:b4:a7:9d:69:6c:ff:71:d6:2d
    Signature Algorithm: sha256WithRSAEncryption
        Issuer: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=Red Hat Consulting Root CA
        Validity
            Not Before: Mar 23 10:59:32 2021 GMT
            Not After : Mar 21 10:59:32 2031 GMT
        Subject: C=ES, ST=Catalunya, L=Barcelona, O=Red Hat Consulting, OU=Red Hat Consulting Certificate Authority, CN=lbtower.bcnconsulting.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:b5:33:af:52:d2:15:bb:6b:77:ff:f2:67:58:2e:
                    ...
                    df:5c:5b:20:21:2b:e9:d9:85:58:cf:c2:3d:80:bf:
                    ca:d7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Alternative Name: 
                DNS:lbtower.bcnconsulting.com, DNS:tower1.bcnconsulting.com, DNS:tower2.bcnconsulting.com, DNS:tower3.bcnconsulting.com
            X509v3 Subject Key Identifier: 
                DA:06:CC:76:89:49:FF:4F:07:AC:40:2F:25:39:30:0E:6B:36:26:E5
            X509v3 Authority Key Identifier: 
                keyid:D9:AE:54:D3:A1:51:C1:7C:46:34:4C:AE:56:E0:B4:AC:E4:B7:A2:7E

    Signature Algorithm: sha256WithRSAEncryption
         8d:13:c4:26:30:9c:9f:09:9e:10:78:fd:18:10:85:85:19:b8:
         ...
         ad:21:e5:9c:73:a3:8b:48
```


To verify the end-entity certificate we should use the complete chain certificate, if we don't it will fail.   

```bash
$ openssl verify -CAfile /tmp/ca/intermediateCA/certs/ca-intermediate.pem /tmp/ca/endEntity/certs/tower.pem 
/tmp/ca/endEntity/certs/tower.pem: C = ES, ST = Catalunya, L = Barcelona, O = Red Hat Consulting, OU = Red Hat Consulting Certificate Authority, CN = lbtower.bcnconsulting.com
error 20 at 0 depth lookup:unable to get local issuer certificate
      
$ openssl verify -CAfile /tmp/ca/intermediateCA/certs/ca-chain.pem /tmp/ca/endEntity/certs/tower.pem 
/tmp/ca/endEntity/certs/tower.pem: OK         
```

# ToDo
- Multiple Intermediate

# Sources
* [How To create root pair](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html)
* [Frequently_used_OpenSSL_Commands](https://www.xolphin.com/support/OpenSSL/Frequently_used_OpenSSL_Commands)
* [openssl_privatekey](https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html)
* [openssl_csr_module](https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_csr_module.html)
* [openssl_certificate_module](https://docs.ansible.com/ansible/2.7/modules/openssl_certificate_module.html)
