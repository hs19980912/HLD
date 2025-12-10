
## SQS - Competing consumers model

Multiple listeners (consumers) can listen to the same SQS queue — but each message will be delivered to only one of them. This is called the **competing consumers model**.  

It is not fan-out like Kafka.