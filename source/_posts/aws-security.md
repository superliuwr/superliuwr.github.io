---
title: AWS Security
date: 2018-06-16 12:54:01
categories:
- Cloud
- DevOps
tags:
- AWS
---
# AWS Identity and Access Management (IAM)

IAM is a powerful service that allows you to control how people and programs are allowed to manipulate your AWS infrastructure.

IAM is not:
* an identity store/authorization system for your applications
  * use AWS Directory Service or Cognito instead
* operating system identity management
  * shared responsibility

## Principals

### Root user

it has full privileges to do anything in the account, including closing the account. It is strongly recommended that you do not use the root user for your everyday tasks, even the administrative ones.

### IAM users

no expiration period

### ROLES/TEMPORARY SECURITY TOKENS

Roles are used to grant specific privileges to specific **actors** for a set duration of time. These actors can be authenticated by AWS or some trusted external system. When one of these actors assumes a role, AWS provides the actor with a **temporary security token** from the AWS Security Token Service (STS) that the actor can use to access AWS Cloud services. Requesting a temporary security token requires specifying how long the token will exist before it expires. The range of a temporary security token lifetime is 15 minutes to 36 hours.

Roles and temporary security tokens enable a number of use cases:

* Amazon EC2 Roles
  * Granting permissions to applications running on an Amazon EC2 instance.
* Cross-Account Access
  * Granting permissions to users from other AWS accounts, whether you control those accounts or not.
* Federation
  * Granting permissions to users authenticated by a trusted external system.

#### Amazon EC2 Roles

Suppose that an application running on an Amazon EC2 instance needs to access an Amazon Simple Storage Service (Amazon S3) bucket. An IAM role that grants the required access to the Amazon S3 bucket. When the Amazon EC2 instance is launched, the role is assigned to the instance. When the application running on the instance uses the Application Programming Interface (API) to access the Amazon S3 bucket, it assumes the role assigned to the instance and obtains a temporary token that it sends to the API. The process of obtaining the temporary token and passing it to the API is handled automatically by most of the AWS SDKs, allowing the application to make a call to access the Amazon S3 bucket without worrying about authentication.

#### Crosss-account Access

You can set up an IAM role with the permissions you want to grant to users in the other account, then users in the other account can assume that role to access your resources.

#### Federation

IAM Identity Providers provide the ability to federate these outside identities with IAM and assign privileges to those users authenticated outside of IAM.

* OpenID Connect
* SAML

## Authentication

* User name/Password
* Access Key ID/Access Secret Key
* Access Key ID/Access Secret Key/Session Token
  * When a process operates under an assumed role, the temporary security token provides an access key for authentication. In addition to the access key (remember that it consists of two parts), the token also includes a session token. Calls to AWS must include both the two-part access key and the session token to authenticate.

## Authorization

Authorization is handled in IAM by defining specific privileges in policies and associating those policies with principals.

### POLICIES

A policy is a JSON document that fully defines a set of permissions to access and manipulate AWS resources. Policy documents contain one or more permissions, with each permission defining:

* Effect—A single word: Allow or Deny.
* Service—For what service does this permission apply? Most AWS Cloud services support granting access through IAM, including IAM itself.
* Resource—The resource value specifies the specific AWS infrastructure for which this permission applies. This is specified as an Amazon Resource Name (ARN). The format for an ARN varies slightly between services, but the basic format is: "arn:aws:service:region:account-id:[resourcetype:]resource"
* Action—The action value specifies the subset of actions within a service that the permission allows or denies. 
* Condition—The condition value optionally defines one or more additional restrictions that limit the actions allowed by the permission.

A Sample policy:
``` json
{
    "Version": "2012–10–17",
    "Statement": [
        {
            "Sid": "Stmt1441716043000",
            "Effect": "Allow",	<-  This policy grants access
            "Action": [	<-  Allows identities to list
                "s3:GetObject",	<-  and get objects in
                "s3:ListBucket"	<-  the S3 bucket
            ],
            "Condition": {
                "IpAddress": {				<-  Only from a specific
                    "aws:SourceIp": "192.168.0.1"	<-  IP Address
                }
            },
            "Resource": [
                "arn:aws:s3:::my_public_bucket/*"	<-  Only this bucket
            ]
        }
    ]
}
```

### ASSOCIATING POLICIES WITH PRINCIPALS

A policy can be associated directly with an IAM user in one of two ways:

* User Policy—These policies exist only in the context of the user to which they are attached. In the console, a user policy is entered into the user interface on the IAM user page.
* Managed Policies—These policies are created in the Policies tab on the IAM page (or through the CLI, and so forth) and exist independently of any individual user. In this way, the same policy can be associated with many users or groups of users. There are a large number of predefined managed policies that you can review on the Policies tab of the IAM page in the AWS Management Console. In addition, you can write your own policies specific to your use cases.

The other common method for associating policies with users is with the IAM groups feature. Groups simplify managing permissions for large numbers of users. After a policy is assigned to a group, any user who is a member of that group assumes those permissions.

There are two ways a policy can be associated with an IAM group:

* Group Policy—These policies exist only in the context of the group to which they are attached. In the AWS Management Console, a group policy is entered into the user interface on the IAM Group page.
* Managed Policies—In the same way that managed policies (discussed in the “Authorization” section) can be associated with IAM users, they can also be associated with IAM groups.

The final way an actor can be associated with a policy is by assuming a role. In this case, the actor can be:

* An authenticated IAM user (person or process). In this case, the IAM user must have the rights to assume the role.
* A person or process authenticated by a trusted service outside of AWS, such as an on-premises LDAP directory or a web authentication service. In this situation, an AWS Cloud service will assume the role on the actor’s behalf and return a token to the actor.

After an actor has assumed a role, it is provided with a temporary security token associated with the policies of that role. The token contains all the information required to authenticate API calls. This information includes a standard access key plus an additional session token required for authenticating calls under an assumed role.

## Other Key Features

### MULTI-FACTOR AUTHENTICATION (MFA)

## ROTATING KEYS

## RESOLVING MULTIPLE PERMISSIONS

1. Initially the request is denied by default.
2. All the appropriate policies are evaluated; if there is an explicit “deny” found in any policy, the request is denied and evaluation stops.
3. If no explicit “deny” is found and an explicit “allow” is found in any policy, the request is allowed.
4. If there are no explicit “allow” or “deny” permissions found, then the default “deny” is maintained and the request is denied.

The only exception to this rule is if an AssumeRole call includes a role and a policy, the policy cannot expand the privileges of the role (for example, the policy cannot override any permission that is denied by default in the role).

