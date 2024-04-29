# PaaS CDN Broker Bosh Release

This is a Bosh release for deploying [cloud.gov's CDN broker](https://github.com/cloud-gov/cf-cdn-service-broker) on a Bosh-managed VM. We maintain [a fork of the broker](https://github.com/alphagov/paas-cdn-broker) and submodule it into this repository. It is important to note we have kept the submodule path of the upstream repository in order to not have to re-write Golang import paths.

# Deploy with BOSH

## Create infrastructure

The broker requires:

* database
* S3 bucket
* ELB
* security group to access the database

For example you can use terraform to set this up. The BOSH properties below refer to these components.

## BOSH release

```
releases:
  - name: cdn-broker
    version: 0.0.123
    url: https://s3-eu-west-1.amazonaws.com/gds-paas-build-releases/cdn-broker-0.0.123.tgz
    sha1: dddf38bc5d64ec35a920709726e39e9327c177c0
```

## BOSH job

```
jobs:
  - name: cdn_broker
    azs: [z1, z2]
    instances: 2
    vm_type: cdn_broker
    stemcell: default
    templates:
      - name: cdn-broker
        release: cdn-broker
    networks:
      - name: cf
    vm_extensions:
      - cdn_rds_client_sg
```

## BOSH VM type

Required to use specific instance profile, elb and security groups.

```
---
vm_types:
  - name: cdn_broker
    network: cf
    env: cf
    cloud_properties:
      instance_type: t2.nano
      iam_instance_profile: cdn-broker
      ephemeral_disk:
        size: 10240
        type: gp2
      security_groups:
        - cdn_broker_db_clients_security_group
        - default_security_group
      elbs:
        - cdn_broker_elb
```

## BOSH properties

```
properties:
  cdn-broker:
    broker_username: "cdn-broker"
    broker_password: xxx
    database_url: postgresql://xxx
    email: "team@domain.com"
    acme_url: "https://acme-v01.api.letsencrypt.org/directory"
    bucket: cdn_acme_challenge
    iam_path_prefix: letsencrypt
    cloudfront_prefix: cdn
    aws_access_key_id: ""
    aws_secret_access_key: ""
    aws_default_region: "eu-west-1"
    api_address: "https://api.cloudfoundry.com
    client_id: "cdn_broker"
    client_secret: xxx
    default_origin: apps.cloudfoundry.com
```
