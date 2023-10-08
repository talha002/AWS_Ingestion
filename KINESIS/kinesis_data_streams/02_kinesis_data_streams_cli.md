## Connect to CLI (You can check my AWS_Basics repo if you don't know AWS CLI)

## Create data stream
```commandline
aws kinesis create-stream --stream-name KinesisTrain
```
## List data streams
```
aws kinesis list-streams
{
    "StreamNames": [
        "KinesisTrain"
    ],
    "StreamSummaries": [
        {
            "StreamName": "KinesisTrain",
            "StreamARN": "arn:aws:kinesis:eu-central-1:152100380489:stream/KinesisTrain",
            "StreamStatus": "ACTIVE",
            "StreamModeDetails": {
                "StreamMode": "ON_DEMAND"
            },
            "StreamCreationTimestamp": "2023-07-15T22:19:46+03:00"
        }
    ]
}
```

## Describe data stream
```
aws kinesis describe-stream-summary --stream-name KinesisTrain
{
    "StreamDescriptionSummary": {
        "StreamName": "KinesisTrain",
        "StreamARN": "arn:aws:kinesis:eu-central-1:152100380489:stream/KinesisTrain",
        "StreamStatus": "ACTIVE",
        "StreamModeDetails": {
            "StreamMode": "ON_DEMAND"
        },
        "RetentionPeriodHours": 24,
        "StreamCreationTimestamp": "2023-07-15T22:19:46+03:00",
        "EnhancedMonitoring": [
            {
                "ShardLevelMetrics": []
            }
        ],
        "EncryptionType": "NONE",
        "OpenShardCount": 4,
        "ConsumerCount": 0
    }
}
```

## Produce a record
```
aws kinesis put-record --stream-name KinesisTrain --partition-key 123 --data testdata
{
    "ShardId": "shardId-000000000000",
    "SequenceNumber": "49642672777078853941942291364627014396100010722721792002"
}
```
## Consume records
- First you need to get SHARD_ITERATOR then consume records
```
SHARD_ITERATOR=$(aws kinesis get-shard-iterator --shard-id shardId-000000000000 --shard-iterator-type TRIM_HORIZON --stream-name KinesisTrain --query 'ShardIterator')

aws kinesis get-records --shard-iterator $SHARD_ITERATOR
{
    "Records": [
        {
            "SequenceNumber": "49642672777078853941942291364627014396100010722721792002",
            "ApproximateArrivalTimestamp": "2023-07-15T22:21:51.763000+03:00",
            "Data": "testdata",
            "PartitionKey": "123"
        }
    ],
    "NextShardIterator": "AAAAAAAAAAEM4v0SUdMVAYQ3A+BWzzqDnjTCDlC0/qJptRM614c8EfKjkAPPZwO485TQKay4YixZBjFhBvp0MMVe4vLuAvwS12tStvL92uf+DwUxowr9P15IoPRr8yQo1x3VTwT7PKYxv0RSwdCttHQJGSXjFRDrRGddvyQYCkQcYV6dnpmTnH/dIxRaek28RW56UKCwDWaslb1bR7CWT++pbbxm2VmUtkS5TJjIQuMICeAjylNEdlzqXZpDs9Lcv+8Es/Ic1P0=",
    "MillisBehindLatest": 0
}
```
## Delete data streams
```
aws kinesis delete-stream --stream-name KinesisTrain
```

## List if it is deleted
```
aws kinesis list-streams
{
    "StreamNames": [],
    "StreamSummaries": []
}
```