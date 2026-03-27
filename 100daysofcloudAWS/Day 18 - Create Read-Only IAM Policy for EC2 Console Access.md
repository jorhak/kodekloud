```
When establishing infrastructure on the AWS cloud, Identity and Access Management (IAM) is among the first and most critical services to configure. IAM facilitates the creation and management of user accounts, groups, roles, policies, and other access controls. The Nautilus DevOps team is currently in the process of configuring these resources and has outlined the following requirements.

Create an IAM policy named iampolicy_siva in us-east-1 region, it must allow read-only access to the EC2 console, i.e this policy must allow users to view all instances, AMIs, and snapshots in the Amazon EC2 console.



Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)

Console URL	https://265164650170.signin.aws.amazon.com/console?region=us-east-1
Username	kk_labs_user
Password	contra
Start Time	Wed Mar 04 01:47:21 UTC 2026
End Time	Wed Mar 04 02:47:21 UTC 2026

Notes:

Create the resources only in us-east-1 region.

To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:
toggle button
```

Variables de entorno:
```
IAM_POLICY_NAME=iampolicy_siva
REGION=us-east-1
```

1. Crear **policy.json**:
```
vi policy.json
```

```json
{
   "Version":"2012-10-17",		 	 	 
   "Statement": [{
      "Effect": "Allow",
      "Action": [
         "ec2:DescribeInstances",
         "ec2:DescribeSnapshots"
	  ],
	  "Resource": "*"
   }
   ]
}

```

2. Crear **policy**:
```
aws iam create-policy \
    --policy-name $IAM_POLICY_NAME \
    --policy-document file://policy.json
```

3. Verificar **policy**:
```
aws iam list-policies

aws iam get-policy \
--policy-arn arn:aws:iam::265164650170:policy/iampolicy_siva
```
