# myStrom prometheus exporter

Export [myStrom WiFi Switch](https://mystrom.ch/de/wifi-switch-ch/) report
statistics to [Prometheus](https://prometheus.io).

Metrics are retrieved using the [switch REST api](https://api.mystrom.ch/).
This has only be testet using a WiFi switch with firmware 3.82.60, but should be
backwards compatible with all 3.x firmwares.

To run it:
```bash
$ go build
$ ./mystrom-exporter [flags]
```

## Build instructions
The package uses `stringer` to generate `String()` methods on structs, to build the package you need to install `stringer` through `gotools`.

```bash
$ go install golang.org/x/tools/cmd/stringer@latest
# optional, should also be triggered by go build
$ go generate ./...
```

## Exported Metrics
| Metric | Description |
| ------ | ------- |
| mystrom_up | Was the last REST api call to the switch successful |
| mystrom_report_watt_per_sec | The average of energy consumed per second from last call this request |
| mystrom_report_temperatur  | The currently measured temperature by the switch. (Might initially be wrong, but will automatically correct itself over the span of a few hours) |
| mystrom_report_relay | The current state of the relay (wether or not the relay is currently turned on) |
| mystrom_report_power  | The current power consumed by devices attached to the switch |

This exporter receives requests with a target device (myStrom device), pings the WIFI switch and then returns metrics about this device.
With a single exporter, one can monitor several myStrom devices as the same endpoint can be used for multiple myStrom devices.
See below the Prometheus configuration sample.

## Flags
```bash
$ ./mystrom-exporter --help
```
| Flag | Description | Default |
| ---- | ----------- | ------- |
| web.listen-address | Address to listen on | `:9452` |
| web.metrics-path | Path under which to expose exporters own metrics | `/metrics` |
| web.device-path | Path under which the metrics of the devices are fetched, requires `target` parameter | `/device` |

## Prometheus configuration
A enhancement has been made to have only one exporter which can scrape multiple devices. This is configured in
Prometheus as follows assuming we have 4 mystrom devices and the exporter is running locally on the smae machine as
the Prometheus.
```yaml
 - job_name: mystrom
   scrape_interval: 30s
   metrics_path: /device
   honor_labels: true
   static_configs:
   - targets:
     - '192.168.105.11' # Plase your myStrom device IP here
     - '192.168.105.12'
     - '192.168.105.13'
     - '192.168.105.14'
   relabel_configs: # Initial scrape: http://192.168.105.11/device
     - source_labels: [__address__] # First add a new argument to the request -> example: http://192.168.105.11/device?target=192.168.105.11
       target_label: __param_target # The same address is used for the new parameter
     - target_label: __address__ # Now replace the endpoint address
       replacement: 127.0.0.1:9452 # This is where the exporter listens for metrics -> http://127.0.0.1:9452/device?target=192.168.105.11
```

## Supported architectures
Using the make file, you can easily build for the following architectures, those can also be considered the tested ones:
| OS | Arch |
| -- | ---- |
| Linux | amd64 |
| Linux | arm64 |
| Linux | arm |
| Mac | amd64 |
| Mac | arm64 |

Since go is cross compatible with windows, and mac arm as well, you should be able to build the binary for those as well, but they aren't tested.
The docker image is only built & tested for amd64.

## Packages
Packages are built automatically on release, and container images on push to the main branch.

Take a look at the `Releases` or `Packages` tabs on Github.

### Container images
There is a multiplatform build available here https://github.com/peschmae/exporter-go-mystrom/pkgs/container/exporter-go-mystrom
```
docker pull ghcr.io/peschmae/exporter-go-mystrom:latest
```

## License
MIT License, See the included LICENSE file for terms and conditions.