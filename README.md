[![PyPI status](https://img.shields.io/pypi/status/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/) 

# Apigee X - GCP Services

**This is not an official Google product.**<BR>This implementation is not an official Google product, nor is it part of an official Google product. Support is available on a best-effort basis via GitHub.

***

## Intro

This repository contains a proxy for Apigee X able to call 2 GCP Services: BigQuery (read) only and PubSub (publish).

## Requirement

- A Google Cloud Platform account 
- An Apigee X organization

## Installation

- [Install KVM Admin proxy (option)](https://github.com/apigee/devrel/tree/main/references/kvm-admin-api)
- [Install Apigee GCP SA Shareflow](https://github.com/apigee/devrel/tree/main/references/gcp-sa-auth-shared-flow)
- Deploy this proxy
- Create GCP Service Accounts
- Configure GCP Services (BigQuery and PubSub)
- Configure this proxy
