# DAVe-Deployment

This repository contains the tooling for deploying DAVe application into running Kubernetes cluster.

## Prerequisites

The tooling is written using Ansible. It runs only locally - it doesn't connect to any remote machines. To deploy the application, you need to have following tools installed:
* Ansible
* Kubectl with proper configuration to connect to running Kubernetes cluster
* boto library which will be used for communication with Amazon AWS api_dns
* OpenSSL for generating new SSL keys (only the Let's Encrypt playbook)
* Java for automatic creation of the keys for signing JSON Web Tokens

**Before running the playbook, you have to set the AWS credentials for communication with the AWS services as environment variables.**

## Configuration

The configuration is in `group_vars/all/vars.yaml`. It configures different details of the deployment.

| Variable | Explanation | Example |
|--------|-------------|---------|
| `namespace` | Kubernetes namespace where the DAVe application should be deployed | `dave` |
| `database_namespace` | Kubernetes namespace where the database should be deployed | `database` |
| `dns_zone` | Hosted DNS zone which has to exist in Route53 | `dbg-devops.com` |
| `ui_dns` | Hostname of the UI | `snapshot.dave.dbg-devops.com` |
| `api_dns` | Hostname of the API service | `api.snapshot.dave.dbg-devops.com` |
| `auth_salt` | Autnetication salt used to store passwords | `123456` |
| `jwt_keystore_path` | Java Keystore with keys used to sign JWT tokens | `./jwt.keystore` |
| `jwt_keystore_password` | PAssword for the keystore abobe | `123456` |
| `api_key_path` | Path to the API private key | `./api.key` |
| `api_cert_path` | Path to the API public key | `./api.cert` |
| `ui_key_path` | Path to the UI private key | `./ui.key` |
| `ui_cert_path` | Path to the UI public key | `./ui.cert` |
| `database_name` | Name of the MongoDB database the API should be configured to use | `DAVe` |
| `database_hostname` | MongoDB database hostname | `mongo.database` |
| `database_port` | MongoDB database port | `` |

Following configuration is needed only for signing the certificates with LEt's Encrypt:

| Variable | Explanation | Example |
|--------|-------------|---------|
| `letsencrypt_account_key_path` | Path where the Let's Encrypt account key is / should be created | `./account.key` |
| `acme_directory` | Let's Encrypt ACME directory where we the certificates should be signed (production or staging) | `https://acme-staging.api.letsencrypt.org/directory` |

## Installation

To install the application, run:
```
ansible-playbook install.yaml
```

It will create the Kubernetes resources, Route53 records etc. The definition of the Kubernetes objects is in the templates of different roles. Change the templates if you want to change something. **Before running the installation, check the configuration first.**

## Uninstallation

To uninstall the application, run:
```
ansible-playbook uninstall.yaml
```

It will remove all Kubernetes resources from the cluster as well as the Route53 DNS records. **Before running the uninstallation, check the configuration first.**

## Let's Encrypt

Playbook `letsencrypt.yaml` can be used to obtain signed keys from Let's Encrypt CA:
```
ansible-playbook letsencrypt.yaml
```

* If the account key is missing, it will generate new account key
* If the UI key is missing, it will generate the UI key with the UI hostname in CN
* It will generate UI CSR
* If the API key is missing, it will generate the API key with the API hostname in CN
* It will generate API CSR
* It will try to get the certificates signed. DNS verification will be used - Route53 will be automatically used to handle it.
* The keys will be signed only if the old keys expire in 10 or less days

*The `acme_directory` variable can be used to define whether the production Let's Encrypt service should be used or the staging service (for testing).*
