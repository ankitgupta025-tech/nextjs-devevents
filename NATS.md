
Working locally with messaging systems like RabbitMQ, Kafka, NATS requires quick and easy 
setup on your local machine

Before starting i want to share few key differences between the different brokers

### Rule of thumb
1. Event retention
- If you can delete messages after processing → RabbitMQ / NATS
- If you must never lose history → Kafka

If your application requirement is to build *event-sourcing architecture* Kafka should be your choice 

2. System requirements 
 - for inter service communication go with NATS/RabbitMQ as they both have low learning curve and ease of setup.
 - for large data processing, when you need to process massive streams of data. Go with Kafka
 - For long-running processes and background jobs, when you do not want to burden wep application. Go with RabbitMQ

3. Delivery model pull or push
Push model: Broker sends messages to consumer as they arrive. Hence it is
- Realtime, 
- Negligible  latency
- Can overwhelm slow consumer, if it is unable to handle messages(back pressure)

Pull model: Consumer regularly polls for new messages
- Consumer has more control. It can decide when and at which rate to process
- Has latency as per polling interval

Delivery model support

|Delivery Model | Push | Pull | Summary |
|--|--|--|--|
| NATS | ✅ | ✅| Optmized for both
| RabbitMq | ✅ | -- | Pull model not recomended
| Kafka | ❌| ✅| Supports only pull

NATS has good support for both delivery models whereas RabbitMQ supports Push,Kafka only supports pull
Note: RabbitMQ supports pull model but it is not optimized for hence not recomended

Lets look at NATS setup today.

Windows

### Step 1. Start the NATS Server
[Download nats-server.exe.](https://github.com/nats-io/nats-server/releases/download/v2.10.11/nats-server-v2.10.11-windows-arm64.zip)
I am using following version v2.10.11  

Prepare a config file, name *server.conf* 

jetstream {
  max_file: 0
  max_mem: 256MB
}

Now prepare a .bat file 

nats-server.exe -c server.conf
echo 'nats server added'



Done.
Image

### Step 2. Add stream and consumers

// Stream
nats stream add --storage=memory --subjects="iam.>" --retention=interest --replicas=1 --discard=old "iam-events-stream" --max-msgs=-1  --max-msgs-per-subject=-1 --max-bytes=-1 --max-age=-1 --max-msg-size=-1 --dupe-window=2m0s --no-allow-rollup  --deny-delete --no-deny-purge
echo 'nats stream added'

// Consumer 1
nats consumer add --pull --ack=explicit --replay=instant --wait=30s --deliver=all --max-deliver=20 --filter=iam.account.tenantaccount.created "iam-events-stream" "consumer-usermanagement-tenantaccount-created" --max-pending=0 --no-headers-only --backoff=none

// Consumer 2
nats consumer add --pull --ack=explicit --replay=instant --wait=30s --deliver=all --max-deliver=20 --filter=iam.account.tenantaccount.updated "iam-events-stream" "consumer-usermanagement-tenantaccount-updated" --max-pending=0 --no-headers-only --backoff=none
