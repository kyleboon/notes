List out topics on a broker

`kafkacat -L -b kafka-ttc-keystone.prod.target.com | grep topic`

Finding a specific message on a kafka topic by key

`kafkacat -C -b mybroker:9092 -t mytopic -q -e -o beginning -f '%p %o %k\n' | grep --line-buffered 'myregex`
