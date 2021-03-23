# README

* Ansible Role to create a RootCA + Intermediate + End-entity cert

```bash
┌──────────────────────┐             ┌──────────────────────┐               ┌──────────────────────┐
│                      │    Signed   │                      │     Signed    │                      │
│                      │      by     │                      │       by      │                      │
│        RootCA        │◄────────────┤    IntemediateCA     │◄──────────────┤       End-Entity     │
│                      │             │                      │               │       Certificate    │
│                      │             │                      │               │                      │
└───┬──────────────▲───┘             └──────────────────────┘               └──────────────────────┘
    │              │
    │              │
    │    Signed    │
    └──────────────┘
```

* Playbook has been done with tags to be executed according to your needs.

```bash
$ ansible-playbook tests/test.ym  --tags create-directories,create-end-entity-cert,create-intermediate-ca,create-root-ca
```

* The directory structure created is the following:

```bash
# tree ca/
ca/
├── endEntity
│   ├── certs
│   │   ├── satellite.pem
│   │   ├── tower.pem
│   │   ├── wilcard_bcn.pem
│   │   └── wilcard_mad.pem
│   ├── csr
│   │   ├── satellite.csr
│   │   ├── tower.csr
│   │   ├── wilcard_bcn.csr
│   │   └── wilcard_mad.csr
│   └── private
│       ├── satellite.key
│       ├── tower.key
│       ├── wilcard_bcn.key
│       └── wilcard_mad.key
├── intermediateCA
│   ├── certs
│   │   ├── ca-intermediate.pem
│   │   └── complete_chain.pem
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
```
Useful openssl commands to check certs:

```bash
openssl x509 -text -noout -in /tmp/ca/intermediateCA/certs/complete_chain.pem
openssl req -text -noout -verify -in /tmp/ca/endEntity/csr/wilcard_bcn.csr
openssl verify -CAfile /tmp/ca/intermediateCA/certs/complete_chain.pem /tmp/ca/endEntity/certs/wilcard_mad.pem 
```

# ToDo
- Better README.md
- Multiple Intermediate

# Sources
[How To create root pair](https://jamielinux.com/docs/openssl-certificate-authority/create-the-root-pair.html)
[Frequently_used_OpenSSL_Commands](https://www.xolphin.com/support/OpenSSL/Frequently_used_OpenSSL_Commands)
[openssl_privatekey](https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_privatekey_module.html)
[openssl_csr_module](https://docs.ansible.com/ansible/latest/collections/community/crypto/openssl_csr_module.html)
[openssl_certificate_module](https://docs.ansible.com/ansible/2.7/modules/openssl_certificate_module.html)
