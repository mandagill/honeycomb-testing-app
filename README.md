# README

## Get an API key from Honeycomb

https://docs.honeycomb.io/get-started/configure/environments/manage-api-keys/

You should use an ingest key as these have less privileges than configuration keys.
In our case, our ingest key starts with "hca" and has 64 characters.

```bash
export YOUR_HC_API_KEY="your-secret-key-here"
```

## Start collector via docker

[link to documentation here]

```bash
docker run --rm --name collector-159pm -e YOUR_HC_API_KEY=$YOUR_HC_API_KEY \
  -v ./opentelemetry-collector.yaml:/opentelemetry-collector.yaml \
  -p 127.0.0.1:4317:4317 \
  -p 127.0.0.1:4318:4318 \
  -p 127.0.0.1:55679:55679 \
  otel/opentelemetry-collector:0.99.0 --config=/opentelemetry-collector.yaml
# leave this command running, it'll display the log
```

## Send test telemetry data

https://opentelemetry.io/docs/collector/quick-start/

```bash
telemetrygen traces --otlp-http  --otlp-insecure --otlp-endpoint "localhost:4318" --traces 10
```

You should see things like [get from telemetrygen doc]

```text
Span #19
    Trace ID       : eaf7266466d0903c981518b63cb5c68b
    Parent ID      : 
    ID             : ef773ed9597deb32
```

Also, the counters displayed on http://localhost:55679/debug/tracez should increase as
you generate more traces with the telemetrygen command.

## Start Rails application

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4318"
export OTEL_EXPORTER_OTLP_PROTOCOL="http/json"
export OTEL_EXPORTER_OTLP_TRACES_PROTOCOL="http/json"
bin/rails server
```

Point browser to [http://localhost:3000/posts](http://localhost:3000/posts) and do some activity.
You should see things like

```text
Span #1
    Trace ID       : 99fcc0aa57fb478023b383242082c62c
    Parent ID      : 927dfe074825fbc0
    ID             : a23e03041c974049
    Name           : Post.find_by_sql
    Kind           : Internal
```
