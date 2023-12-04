# home-assistant

Home Assistant

This chart is not maintained by the upstream project

## Source Code

* <https://github.com/home-assistant/home-assistant>
* <https://github.com/damacus/charts/tree/main/charts/application/home-assistant>

## Requirements

Kubernetes: `>=1.16.0-0`

## Dependencies

| Repository | Name |
|------------|------|
| <https://charts.bitnami.com/bitnami> | influxdb |
| <https://charts.bitnami.com/bitnami> | mariadb |
| <https://charts.bitnami.com/bitnami> | postgresql |
| <https://damacus.github.io/charts> | common |

## TL;DR

```shell
helm repo add damacus https://damacus.github.io/charts/
helm repo update
helm install home-assistant damacus/home-assistant
```

## Installing the Chart

To install the chart with the release name `home-assistant`

```shell
helm install home-assistant damacus/home-assistant
```

## Uninstalling the Chart

To uninstall the `home-assistant` deployment

```shell
helm uninstall home-assistant
```

## Configuration

The [values.yaml](./values.yaml) has several commented out suggested values.

Additional values may be used from [values.yaml](https://github.com/damacus/charts/tree/main/charts/applibrary/common/values.yaml) in the [common library](https://github.com/damacus/charts/tree/main/charts/applibrary/common/).

A YAML file that specifies the values for the above parameters can be provided while installing the chart.

```shell
helm install home-assistant damacus/home-assistant -f values.yaml
```

## Custom configuration

### HTTP 400 bad request while accessing from your browser

When configuring Home Assistant behind a reverse proxy make sure you configure the [http](https://www.home-assistant.io/integrations/http) component and set `trusted_proxies` correctly and `use_x_forwarded_for` to `true`.

For example (by edit the configuration.yaml hosted in your pod):

```yaml
http:
  server_host: 0.0.0.0
  ip_ban_enabled: true
  login_attempts_threshold: 5
  use_x_forwarded_for: true
  trusted_proxies:
  # Pod CIDR
  - 10.69.0.0/16
  # Node CIDR
  - 192.168.42.0/24
```

### Z-Wave / Zigbee

A Z-Wave and/or Zigbee controller device could be used with Home Assistant if passed through from the host to the pod. Skip this section if you are using zwave2mqtt and/or zigbee2mqtt or plan to.

First you will need to mount your Z-Wave and/or Zigbee device into the pod, you can do so by adding the following to your values:

```yaml
persistence:
  usb:
    enabled: true
    type: hostPath
    hostPath: /path/to/device
```

Second you will need to set a nodeAffinity rule, for example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: app
          operator: In
          values:
          - zwave-controller
```

... where a node with an attached zwave and/or zigbee controller USB device is labeled with `app: zwave-controller`

### Websockets

If an ingress controller is being used with Home Assistant, web sockets must be enabled using annotations to enable support of web sockets.

Using NGINX as an example the following will need to be added to your values:

```yaml
ingress:
  main:
    enabled: true
    annotations:
      nginx.org/websocket-services: home-assistant
    hosts:
      - host: home-assistant.example.org
        paths:
          - path: /
```

The value derived is the name of the kubernetes service object for home-assistant

### Metrics collection

If metrics collection is enabled through the `metrics.enabled: true` setting, make sure to also enable the Prometheus
endpoint in your Home-Assistant configuration. See the [official documentation](https://www.home-assistant.io/integrations/prometheus/) for more details on how to set this up.
