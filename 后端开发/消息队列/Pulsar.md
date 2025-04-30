# Pulsar

## docker 启动

```shell
docker run -it \
-p 6650:6650 \
-p 8080:8080 \
--mount source=pulsardata,target=/pulsar/data \
--mount source=pulsarconf,target=/pulsar/conf \
-e PULSAR_MEM="-Xms256m -Xmx512m -XX:MaxDirectMemorySize=256m" \
apachepulsar/pulsar:4.0.3 \
bin/pulsar standalone
```

