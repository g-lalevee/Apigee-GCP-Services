[![PyPI status](https://img.shields.io/pypi/status/ansicolortags.svg)](https://pypi.python.org/pypi/ansicolortags/) 

# Apigee - GCP Services

**This is not an official Google product.**<BR>This implementation is not an official Google product, nor is it part of an official Google product. Support is available on a best-effort basis via GitHub.

***

## Intro

This repository contains a proxy for Apigee Edge/x/hybrid to call 4 GCP Services: 
- BigQuery (read)
- Cloud Firestore in Native mode (list), 
- Cloud Logging (write)
- PubSub (publish). 

It doesn't use Apigee Edge extension features.

![Proxy Overview](/images/proxy-overview.jpg)

## Requirement

- A Google Cloud Platform account 
- An Apigee organization (Edge or X/hybrid)

## Installation

1. [Install KVM Admin proxy (option)](https://github.com/apigee/devrel/tree/main/references/kvm-admin-api)<BR>This proxy is required with Apigee X/hybrid to create KVM entry. With Apigee Edge, you can use Apigee Web UI.

1. [Install Apigee Sackmesser (option)](https://github.com/apigee/devrel/tree/main/tools/apigee-sackmesser)<BR>The Apigee Sackmesser lets you deploy API proxies, shared flows and configuration to Apigee Edge as well as hybrid/X without writing any additional manifest files..

1. [Install Apigee GCP SA Shareflow](https://github.com/apigee/devrel/tree/main/references/gcp-sa-auth-shared-flow)<BR>This sharedflow is used obtain access tokens for Google Cloud service accounts. Access tokens are cached in a dedicated environment cache resource for 10min, and used to call GCP services.

1. Deploy this proxy<BR>Deploy proxy in **apiproxy** folder.

1. Create GCP Service Accounts<BR>To authorize Apigee to use Google Cloud BigQuery, Firestore, Cloud Logging and PubSub, you must first: 
    - Create 4 service accounts in Google Cloud and assign it the necessary roles to access your BigQuery dataset, your Firestore collection and write in Cloud Logging and publish a message into a PubSub topic (see [Understanding GCP roles](https://cloud.google.com/iam/docs/understanding-roles)).  
    - Create and download the 4 json keys for the 3 service accounts

1. Configure GCP Services (BigQuery, Cloud Firestore in Native mode and PubSub)<BR>If needed, in your GCP project, create a BigQuery table, a Firestore Collection and a PubSub topic to be able to use this proxy.

1. Upload your 4 GCP Service Account JSON files content into KVM Entries<BR>If you use Apigee X/hybrid, you can create a KVM using the following command

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

After that, to store BigQuery, Firestore, Cloud Logging and PubSub Service accounts JSON file content, create KVM entries using kvm-admin-api:

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
   -d '{ "key": "MyFSN__privKeyPem", "value": "<copy here Firestore SA key file jSON content>" } ' 

curl -i -X POST \
   "https://$APIGEE_HOSTNAME/kvm-admin/v1/organizations/${APIGEE_ORG}/environments/$APIGEE_ENV/keyvaluemaps/$KVM_NAME/entries"
   -H "Content-Type:application/json" \
   -H "Authorization:Bearer $TOKEN" \
   -d '{ "key": "MyLOG__privKeyPem", "value": "<copy here Cloud Logging SA key file jSON content>" } '

curl -i -X POST \
   "https://$APIGEE_HOSTNAME/kvm-admin/v1/organizations/${APIGEE_ORG}/environments/$APIGEE_ENV/keyvaluemaps/$KVM_NAME/entries"
   -H "Content-Type:application/json" \
   -H "Authorization:Bearer $TOKEN" \
   -d '{ "key": "MyPubSub__privKeyPem", "value": "<copy here PubSub SA key file jSON content>" } ' 
```
If you use Apigee Edge, you can also use the Web UI.

1. Deploy this proxy to your Apigee organization

    Exemple: straight deployment from Github to Apigee X / hybrid using [Sackmesser](https://github.com/apigee/devrel/tree/main/tools/apigee-sackmesser).
    ```
    export TOKEN=$(gcloud auth print-access-token)
    export APIGEE_ORG=<YOUR-ORG-NAME>
    export APIGEE_ENV=<YOUR-ENV-NAME>

    sackmesser deploy -g https://github.com/g-lalevee/Apigee-GCP-Services \
    --googleapi \
    -t "$TOKEN" \
    -o "$APIGEE_ORG" \
    -e "$APIGEE_ENV" 
    ```

1. Configure this proxy
    1. If needed, update AM.GCPScopes.BQ, AM.GCPScopes.Firestore, AM.GCPScopes.LOG and AM.GCPScopes.PubSub policies to set the required scopes. In, this sample proxy, we will use: 
    - ```https://www.googleapis.com/auth/pubsub``` scope for PubSub, 
    -  ```https://www.googleapis.com/auth/datastore``` for Firestore  
    -  ```https://www.googleapis.com/auth/logging.write``` for Cloud Logging
    - ```https://www.googleapis.com/auth/bigquery.readonly``` for BigQuery.
    <BR>You can find documentation about GCP service scopes in [GCP documentation.](https://developers.google.com/identity/protocols/oauth2/scopes)

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
    ```

    3. Update AM.setHeaderPayload.Firestore policy<BR>Set **gcp.projectid** and **collection.name** value with your GCP Project ID and Firestore collection name:
    
    ```
    <AssignVariable>
        <Name>gcp.projectid</Name>
        <Value>xxxxx</Value>
    </AssignVariable>
    <AssignVariable>
        <Name>collection.name</Name>
        <Value>yyyyy</Value>
    </AssignVariable>
    ```

    4. Update AM.setHeaderPayload.LOG policy<BR>Set **gcp.projectid** and **gcplogging.logid** value with your GCP Project ID and the log name you want to use:
    
    ```
    <AssignVariable>
        <Name>gcp.projectid</Name>
        <Value>xxxxx</Value>
    </AssignVariable>
    <AssignVariable>
        <Name>gcplogging.logid</Name>
        <Value>yyyyy</Value>
    </AssignVariable>
    ```
    > **_NOTE:_**  the log record format is defined in JS-SetLoggingRecord javascript policy

    5. AM.setHeaderPayload.PubSub policy.<BR>Set **gcp.projectid** and **gcp.topicid** value with your GCP Project ID and PubSub topic:

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
1. Save and deploy proxy

## Test

You can now test your proxy.
- BigQuery (no need to specify query, query is embeded in proxy itself)
```
curl -i -X GET  'https://<YOUR-HOSTNAME>/v1/gcpservice/bq'
```

- Firestore (it returns all documents in the collection defined in proxy)
```
curl -i -X GET 'https://<YOUR-HOSTNAME>/v1/gcpservice/fs'
```

- Cloud Logging (it returns th elink to Google Cloud Console, Cloud Logging Query builder)
```
curl -i -X GET 'https://<YOUR-HOSTNAME>/v1/gcpservice/log'
```

- PubSub (body contains the message to be post in topic specified in the proxy)
``` 
curl -i -X POST \
   -H "Content-Type:application/json" \
   -d 'hello world!'  'https://<YOUR-HOSTNAME>/v1/gcpservice/ps'
```


Et voilà !!