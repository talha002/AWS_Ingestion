- Blog: https://docs.aws.amazon.com/msk/latest/developerguide/serverless-getting-started.html

![](images/02_ec2_amazon_kafka_produce_consume.drawio.png)

---

## Step 1: Create an MSK Serverless cluster
- Amazon MSK service -> Create cluster
- Creation method: `Quick create`
- Cluster name: `msk-serverless-tutorial-cluster`
- Cluster type: `Serverless`
- Copy some important information:
  - VPC: vpc-0769ee73f0727dbf2 (default) 
  - Subnets:	
    - subnet-0606246ffb909824f 
    - subnet-063d9ab17e6a99bb2 
    - subnet-06d1a3540638edd58
  - Security groups associated with VPC: sg-0f113218f8740d188
- Click Create cluster

## Step 2: Create an IAM role
### 2.1. Create policy
- IAM Service -> Policies -> Create Policy
- Policy editor: JSON
- Paste following 
  - Replace `region` with your cluster region for example eu-central-1. 
  - Replace `Account-ID` with your account ID. It is a number right top corner copy it. 
  - Replace `msk-serverless-tutorial-cluster` with the name of your cluster.
```commandline
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kafka-cluster:Connect",
                "kafka-cluster:AlterCluster",
                "kafka-cluster:DescribeCluster"
            ],
            "Resource": [
                "arn:aws:kafka:<region>:<Account-ID>:cluster/<cluster-name>/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "kafka-cluster:*Topic*",
                "kafka-cluster:WriteData",
                "kafka-cluster:ReadData"
            ],
            "Resource": [
                "arn:aws:kafka:<region>:<Account-ID>:topic/<cluster name>/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "kafka-cluster:AlterGroup",
                "kafka-cluster:DescribeGroup"
            ],
            "Resource": [
                "arn:aws:kafka:<region>:<Account-ID>:group/<cluster-name>/*"
            ]
        }
    ]
}
```
- Name: `msk-serverless-tutorial-policy`
### 2.2 Create role
- IAM -> Roles -> Create role
- Common use cases, choose EC2, then choose Next: Permissions
- Search and Select `msk-serverless-tutorial-policy`
- Name: `msk-serverless-tutorial-role`
- Create role
## Step 3: Create a client machine
- EC2 -> Create instance 
- Name: `msk-serverless-tutorial-client`
- Advanced details: IAM instance profile -> `msk-serverless-tutorial-role`
- Create instance

- SSH to Instance
- Install java
```commandline
sudo yum -y install java-11
```
- Download and extract kafka binaries
```commandline
wget https://archive.apache.org/dist/kafka/2.8.1/kafka_2.12-2.8.1.tgz
tar -xzf kafka_2.12-2.8.1.tgz
```
- Go to the kafka_2.12-2.8.1/libs directory, then run the following command to download the Amazon MSK IAM JAR file. The Amazon MSK IAM JAR makes it possible for the client machine to access the cluster.
```commandline
wget https://github.com/aws/aws-msk-iam-auth/releases/download/v1.1.1/aws-msk-iam-auth-1.1.1-all.jar
```
- Go to the kafka_2.12-2.8.1/bin directory. Copy the following property settings and paste them into a new file. Name the file `client.properties` and save it.
```properties
security.protocol=SASL_SSL
sasl.mechanism=AWS_MSK_IAM
sasl.jaas.config=software.amazon.msk.auth.iam.IAMLoginModule required;
sasl.client.callback.handler.class=software.amazon.msk.auth.iam.IAMClientCallbackHandler
```
## Step 4: Create an Apache Kafka topic
- Learn cluster endpoint: 
In the Cluster summary section, choose View client information. This button remains grayed out until Amazon MSK finishes creating the cluster. You might need to wait a few minutes until the button becomes active so you can use it.

Copy the string under the label Endpoint. This is your bootstrap server string.
- go to the kafka_2.12-2.8.1/bin
```properties
export BS=my-endpoint
```
- Create topic
```commandline
./kafka-topics.sh --bootstrap-server $BS \
--command-config client.properties \
--create --topic msk-serverless-tutorial --partitions 6
```


## Step 5: Produce and consume data
- Producer (first terminal)
```commandline
./kafka-console-producer.sh --broker-list $BS \
--producer.config client.properties \
--topic msk-serverless-tutorial
```
- Open another terminal and SSH to EC2 machine for consumer
```commandline
export BS=my-endpoint
./kafka-console-consumer.sh --bootstrap-server $BS \
--consumer.config client.properties \
--topic msk-serverless-tutorial --from-beginning
```
## Step 6: Delete resources
- Delete kafka cluster
- Delete EC2 Instance
- Delete role
- Delete policy