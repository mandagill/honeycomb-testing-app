# README

## How to setup Honeycomb in a Rails application

This example is based
on the [Quick start guide to setup the OpenTelemetry collector](https://opentelemetry.io/docs/collector/quick-start/).

To install Rails follow the [Getting Started with Rails Guide](https://guides.rubyonrails.org/getting_started.html).

The included application was generated using
```bash
bin/rails generate scaffold Post name:string title:string content:text
```

## Get an API key from Honeycomb

Follow the [Honeycomb documentation](https://docs.honeycomb.io/get-started/configure/environments/manage-api-keys/).

You should use an ingest key as these have less privileges than configuration keys.
In our case, our ingest key starts with "hca" and has 64 characters.

```bash
export YOUR_HC_API_KEY="your-secret-key-here"
```

## Start collector via docker

```bash
export YOUR_HC_API_KEY="your-secret-key-here"
docker run \
  --rm \
  --name collector-159pm  \
  -e YOUR_HC_API_KEY=$YOUR_HC_API_KEY \
  -v ./opentelemetry-collector.yaml:/opentelemetry-collector.yaml \
  -p 127.0.0.1:4317:4317 \
  -p 127.0.0.1:4318:4318 \
  -p 127.0.0.1:55679:55679 \
  otel/opentelemetry-collector:0.99.0 \
  --config=/opentelemetry-collector.yaml
# leave this command running, it'll display the log
```

## Send test telemetry data

`telemetrygen` is a utility written in go that simulates a client generating OpenTelemetry traces, metrics, and logs
([documentation](https://github.com/open-telemetry/opentelemetry-collector-contrib/tree/main/cmd/telemetrygen)).

```bash
~/go/bin/telemetrygen traces --otlp-http  --otlp-insecure --otlp-endpoint "localhost:4318" --traces 10
```

You should see things like this in the output for the `opentelemetry-collector` container running in docker.

```text
Span #19
    Trace ID       : eaf7266466d0903c981518b63cb5c68b
    Parent ID      : 
    ID             : ef773ed9597deb32
```

Also, the counters displayed on http://localhost:55679/debug/tracez should increase as
you generate more traces with the telemetrygen command.

The dashboard on https://ui.honeycomb.io should display data for the `telemetrygen` dataset.

## Start Rails application

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT="http://localhost:4318"
export OTEL_EXPORTER_OTLP_PROTOCOL="http/protobuf"
export OTEL_SERVICE_NAME="testing-app"
bin/rails server
```

Point a browser to http://localhost:3000/posts and do some operations on post: show/create/edit/destroy.
You should see things like this in the output for the `opentelemetry-collector` container running in docker.

```text
Span #1
    Trace ID       : 99fcc0aa57fb478023b383242082c62c
    Parent ID      : 927dfe074825fbc0
    ID             : a23e03041c974049
    Name           : Post.find_by_sql
    Kind           : Internal
```

Also, the counters displayed on http://localhost:55679/debug/tracez should increase as
you perform more actions.

The dashboard on https://ui.honeycomb.io should display data for the `testing-app` dataset.
