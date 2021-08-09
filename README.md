[![PyPI status](https://img.shields.io/pypi/status/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/) 

# Apigee - GCP Services

**This is not an official Google product.**<BR>This implementation is not an official Google product, nor is it part of an official Google product. Support is available on a best-effort basis via GitHub.

***

## Intro

This repository contains a proxy for Apigee X able to call GCP Services: BigQuery (read) only and PubSub (publish). It doesn't use Apigee Edge extension features.

## Requirement

- A Google Cloud Platform account 
- An Apigee organization (Edge or X/hybrid)

## Installation

1. [Install KVM Admin proxy (option)](https://github.com/apigee/devrel/tree/main/references/kvm-admin-api)<BR>This proxy is required with Apigee X/hybrid to create KVM entry. With Apigee Edge, you can use Apigee Web UI.

2. [Install Apigee GCP SA Shareflow](https://github.com/apigee/devrel/tree/main/references/gcp-sa-auth-shared-flow)<BR>This sharedflow is used obtain access tokens for Google Cloud service accounts. Access tokens are cached in a dedicated environment cache resource for 10min, and used to call GCP services.

3. Deploy this proxy<BR>Deploy proxy in **apiproxy** folder.

4. Create GCP Service Accounts<BR>To authorize Apigee to use Google Cloud BigQuery and PubSub, you must first: 
    - Create a service account in Google Cloud and assign it the necessary roles to access your BigQuery dataset and publish intot a PubSub Topicc. 
    - Create and download the 2 json keys for the service accounts

5. Configure GCP Services (BigQuery and PubSub)<BR>If needed, in your GCP project, create a BigQuery table and PubSub topic to be able to use this proxy.

6. Upload your 2 GCP Service Account JSON files content into KVM Entries<BR>If you use Apigee X/hybrid, you can create a KVM using the following command

```sh
export TOKEN=$(gcloud auth print-access-token)
export APIGEE_ORG=<my-org-name>
export APIGEE_ENV=<my-env>
export KVM_NAME=GCP-SA-settings

curl -X POST \
    "https://apigee.googleapis.com/v1/organizations/${APIGEE_ORG}/environments/$APIGEE_ENV/keyvaluemaps" \
    -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json" \
    --data "{\"name\":\"$KVM_NAME\",\"encrypted\": true}"
```

After that, to store BigQuery and PubSub Service accounts JSON file content, create KVM entries using kvm-admin-api:

```sh
export TOKEN=$(gcloud auth print-access-token)
export APIGEE_HOSTNAME=<my-apigee-hostname>
export APIGEE_ORG=<my-org-name>
export APIGEE_ENV=<my-env>
export KVM_NAME=GCP-SA-settings

curl -i -X POST \
   "https://$APIGEE_HOSTNAME/kvm-admin/v1/organizations/${APIGEE_ORG}/environments/$APIGEE_ENV/keyvaluemaps/$KVM_NAME/entries"
   -H "Content-Type:application/json" \
   -H "Authorization:Bearer $TOKEN" \
   -d '{ "key": "MyBQ__privKeyPem", "value": "<copy here BigQuery SA key file jSON content>" } ' 

curl -i -X POST \
   "https://$APIGEE_HOSTNAME/kvm-admin/v1/organizations/${APIGEE_ORG}/environments/$APIGEE_ENV/keyvaluemaps/$KVM_NAME/entries"
   -H "Content-Type:application/json" \
   -H "Authorization:Bearer $TOKEN" \
   -d '{ "key": "MyPubSub__privKeyPem", "value": "<copy here PubSub SA key file jSON content>" } ' 
```
If you use Apigee Edge, you can also use the Web UI.

7. Configure this proxy
    1. If needed, update AM.GCPScopes.BQ and AM.GCPScopes.PubSub policies to set the required scopes. In, this sample proxy, we will use ```https://www.googleapis.com/auth/pubsub``` scope for PubSub, and, ```https://www.googleapis.com/auth/bigquery.readonly``` for BigQuery.<BR>You can find documentation about GCP service scopes in [GCP documentation.](https://developers.google.com/identity/protocols/oauth2/scopes)

    2. Update AM.setHeaderPayload.BQ policy<BR>Update the payload with your BigQuery query statement:
    ```
    <Payload contentType="application/json">
            {"query":"SELECT date, airline,airline_code, departure_airport, ....
            FROM [bigquery-samples.airline_ontime_data.flights] LIMIT 10"}
    </Payload>
    ```
    In this sample, we use the dataset **airline_ontime_data** from project **bigquery-samples**, a public data project automatically pinned to every GCP project. <BR>Then, set **gcp.projectid** value with your GCP Project ID:
    ```
    <AssignVariable>
        <Name>gcp.projectid</Name>
        <Value>xxxxx</Value>
    </AssignVariable>
    ````

    3. AM.setHeaderPayload.PubSub policy.<BR>Set **gcp.projectid** and **gcp.topicid** value with your GCP Project ID and PubSub topic:

    ```
    <AssignVariable>
        <Name>gcp.projectid</Name>
        <Value>xxxxx</Value>
    </AssignVariable>
    <AssignVariable>
        <Name>gcp.topicid</Name>
        <Value>yyyyy</Value>
    </AssignVariable>
    ```
8. Save and deploy proxy

## Test

You can now test your proxy.
- BigQuery (no need to specify query, query is embeded in proxy itself)
```
curl -i -X GET  'https://<YOUR-HOSTNAME>/v1/gcpservice/bq'
````

- PubSub (body contains the message to be post in topic specified in the proxy)
``` 
curl -i -X POST \
   -H "Content-Type:application/json" \
   -d 'hello world!'  'https://<YOUR-HOSTNAME>/v1/gcpservice/ps'
```


Et voilà !!