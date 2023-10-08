
![](images/01_cloudshell_firehose_s3_demo_architecture.png)

---

## Create Firehose 
- Amazon Kinesis service -> Data Firehose
- Create delivery system
- Source: `Direct PUT`
- Destination: `Amazon S3`
- Delivery stream name: `FireHoseTrain`
- S3 bucket: <your destination bucket>
- S3 bucket prefix: `firehose_boto3`
- Buffer hints, compression and encryption:
  - Buffer size: 1 MB
  - Buffer interval: 120 secs
- Create delivery system

## Open cloudshell
python3
```python
import boto3
client_firehose = boto3.client('firehose')

for i in range(1000):
	response = client_firehose.put_record(DeliveryStreamName='FireHoseTrain',Record={'Data': f'Message from boto3 - {i} \n'.encode('utf8')})

```


## Go to target bucket 
- File content should be like that
```python
Message from boto3 - 0 
Message from boto3 - 1 
Message from boto3 - 2 
Message from boto3 - 3 
Message from boto3 - 4 
Message from boto3 - 5 
Message from boto3 - 6 
Message from boto3 - 7 
Message from boto3 - 8 
Message from boto3 - 9 
Message from boto3 - 10 
Message from boto3 - 11 
Message from boto3 - 12 
Message from boto3 - 13 
Message from boto3 - 14 
Message from boto3 - 15 
Message from boto3 - 16 
Message from boto3 - 17 
Message from boto3 - 18 
Message from boto3 - 19 
Message from boto3 - 20 
Message from boto3 - 21 
Message from boto3 - 22 
Message from boto3 - 23 
...
...
...
```