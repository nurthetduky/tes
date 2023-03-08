# AWS CodeDeploy
Code and Commands for the AWS CodeDeploy [video](https://www.youtube.com/watch?v=XyP_WcF6Moo).

## Create S3 Artifact Bucket

1. Create s# Bucket (CodeBuild Build Artifacts)
```
aws s3api list-buckets
aws s3api create-bucket --bucket YOUR_BUCKET_NAME --region YOUR_REGION
```

2. Add versioning to Bucket
```
aws s3api put-bucket-versioning --bucket YOUR_BUCKET_NAME --versioning-configuration Status=Enabled --region YOUR_REGION
```

3. Display Bucket
```
aws s3api list-buckets
```

4. Compress Deployment Artifacts and push to Bucket for YOUR_CODE_DEPLOY_APP_NAME CodeBuild Application
	- index.html
	- buildspec.yml
	- /scripts/*.sh


FIRST: Create CodeDeploy Application

```
aws deploy push --application-name YOUR_CODE_DEPLOY_APP_NAME --s3-location s3://YOUR_BUCKET_NAME/YOUR_CODE_DEPLOY_APP_NAME/app.zip --ignore-hidden-files --region YOUR_REGION
```

<hr />

## Create EC2 Instances

1. Get VPC ID
```
aws ec2 describe-vpcs
```

2. Get Subet Id
```
aws ec2 describe-subnets
```

3. Create Security Group
```
aws ec2 create-security-group \
   --group-name cc-sg \
   --description "cc SG" \
   --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=cc-sg}]' \
   --vpc-id "YOUR_VPC_ID"
```

4. Create SG Ingress
	- 4.1 Create SG Ingress (SSH)
```	
aws ec2 authorize-security-group-ingress \
	--group-id YOUR_SG_ID \
	--protocol tcp \
	--port 22 \
	--cidr 0.0.0.0/0 \
	--profile cc_developer 
```
	
4. Create SG Ingress
	- 4.2 Create SG Ingress (HTTP)
```
aws ec2 authorize-security-group-ingress \
	--group-id YOUR_SG_ID \
	--protocol tcp \
	--port 80 \
	--cidr 0.0.0.0/0 \
	--profile cc_developer
```

5. Create Role for EC2 to read from S3 (trustpolicy.json)

6. Create IAM EC2 CodeDeploy Role
```
aws iam create-role --role-name EC2CodeDeployRole --assume-role-policy-document file://trustpolicy.json
```

7. Attach Role Policy
```
aws iam list-policies | grep -i s3ReadOnly 
aws iam attach-role-policy --role-name EC2CodeDeployRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws iam list-attached-role-policies --role-name EC2CodeDeployRole
```

8. Create EC2 Instance Profile
```
aws iam create-instance-profile --instance-profile-name webserverprofile
```

9. Add EC2 CodeDeploy Role to Instance Profile
```
aws iam add-role-to-instance-profile --role-name EC2CodeDeployRole --instance-profile-name webserverprofile
```

10. Create Key Pair
```
aws ec2 create-key-pair --key-name YOUR_KP --output text > ~/.ssh/cc-key
```

11. Get AMI ID for AMZ Linux2
```
aws ec2 describe-images \
	--owners amazon \
	--region YOUR_REGION \
	--filters "Name=name,Values=amzn2-ami-kernel-5.10-hvm-2.0.20230221.0-x86_64-gp2" "Name=root-device-type,Values=ebs"
```

12. Create UserData file (code_deploy_user_data.txt)

13.  Create Instances
```
aws ec2 run-instances \
--image-id ami-AMI_IMAGE_ID \
--count 2 \
--instance-type t2.micro \
--key-name YOUR_KP \
--iam-instance-profile Name=webserverprofile \
--security-group-ids YOUR_SG_ID \
--subnet-id YOUR_SUBNET_ID \
--block-device-mappings "[{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"VolumeSize\":10,\"DeleteOnTermination\":false}}]" \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=webserver}, {Key=Environment,Value=Dev}]' \
--user-data file://code_deploy_user_data.txt 
```	   

14. CURL
```
aws ec2 describe-instances > instances.json
grep PublicIpAddres instances.json
curl YOUR_IP_ADDRESS_1
curl YOUR_IP_ADDRESS_2
```