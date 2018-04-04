# AutoPilot Pattern Base

Docker base image for implementing the Auto Pilot Pattern.

This image includes [ContainerPilot](https://www.joyent.com/containerpilot), [Consul](https://www.consul.io/) and [Prometheus Node Exporter](https://github.com/prometheus/node_exporter). It can be used as a base image using the `FROM` instruction or as part of a multi-stage build.

## Multi-stage

```dockerfile
FROM pdouble16/autopilotpattern-base:latest
FROM mongo:3.6.3

# Copy important bits from the first image
RUN mkdir -p /var/lib/consul /etc/consul
COPY --from=0 /usr/local/bin/* /usr/local/bin/
COPY --from=0 /etc/consul/* /etc/consul/

ENV CONTAINERPILOT="/etc/containerpilot.json5"
...
COPY etc/containerpilot.json5 ${CONTAINERPILOT}
CMD ["/usr/local/bin/containerpilot"]
```

You must provide `containerpilot.json5` to include your services.

## Base

```dockerfile
FROM pdouble16/autopilotpattern-base:latest

...
RUN apk --no-cache add [packages ...]
COPY etc/containerpilot.json5 ${CONTAINERPILOT}
```

This image is based on Alpine so you must use `apk --no-cache add` to install new packages. You must also provide `containerpilot.json5` to include your services.

## Configuration

The following environment variables may be used to configure the agent:

- `CONSUL` (optional) The DNS name for joining the Consul cluster. Defaults to `consul`.
- `CONSUL_DATACENTER_NAME` (optional) The data center to configure in Consul. If not specified the container will attempt to determine if running in Joyent Triton, AWS or Azure.
- `CONSUL_DNS` (optional) Comma separated list of DNS servers for Consul to delegate to if a request is made for a name not ending with `.consul`. This does not affect container DNS resolution. Defaults to Google public name servers `8.8.8.8,8.8.4.4`.
