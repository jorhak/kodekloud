```
The Nautilus DevOps team has some volumes in different regions in their AWS account. They are going to setup some automated backups so that all important data can be backed up on regular basis. For now they shared some requirements to take a snapshot of one of the volumes they have.
Create a snapshot of an existing volume named datacenter-vol in us-east-1 region.
1) The name of the snapshot must be datacenter-vol-ss.
2) The description must be datacenter Snapshot.
3) Make sure the snapshot status is completed before submitting the task.

Use below given AWS Credentials: (You can run the showcreds command on aws-client host to retrieve these credentials)
Console URL
https://143588534152.signin.aws.amazon.com/console?region=us-east-1
Username
kk_labs_user
Password


Start Time
Sun Mar 01 11:45:13 UTC 2026
End Time
Sun Mar 01 12:45:13 UTC 2026


Notes:
Create the resources only in us-east-1 region.
To display or hide the terminal of the AWS client machine, you can use the expand toggle button as shown below:


```

Variables de entorno
```
VOLUME_NAME=datacenter-vol
SNAPSHOT_NAME=datacenter-vol-ss
SNAPSHOT_DESCRIPTION=”datacenter Snapshot”
REGION=us-east-1
``` 

1. Capturar _volumeId_:
```
VOLUME_ID=$(aws ec2 describe-volumes \
       --filters “Name=tag:Name,Values=$VOLUME_NAME” \
       --query “Volumes[].VolumeId” \
       --output text)
```

2. Crear **Snapshot**:
```
aws ec2 create-snapshot \
--volume-id $VOLUME_ID \
--description “$SNAPSHOT_DESCRIPTION” \
--tag-specifications “ResourceType=snapshot,Tags=[{Key=Name,Value=$SNAPSHOT_NAME}]” \
--region $REGION
```

3. Estado de **Snapshot**:
```
aws ec2 describe-snapshots --filters "Name=tag:Name,Values=$SNAPSHOT_NAME" --query "Snapshots[].{ESTADO:State,DESCRIPCION:Description,NOMBRE:Tags[?Key=='Name'].Value | [0]}" --output table
```
