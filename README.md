# WeeWX Metrics

Metrics writes Prometheus text files containing current weather observations
and day, week, month and year aggregates. A web server must expose these files
for Prometheus to scrape; the extension does not start an HTTP listener.

This repository mirrors [Cosmospacedog/weewx-metrics](https://github.com/Cosmospacedog/weewx-metrics).
The installation below uses the separately repaired catalog archive
`0.0.0.post20261022`. A GitHub source-branch download does not include those
packaging repairs. The catalog repair uses the case-sensitive skin directory `Metrics`.
The mirrored source stores its templates under `skins/METRICS`.

Use WeeWX 5.5–5.x and Python 3.10–3.14 for the catalog archive. The skin uses
WeeWX's Cheetah generator and needs no additional Python package.

## Install and expose the files

Install the published archive with WeeWX 5's extension installer:

```sh
weectl extension install https://github.com/weewx-packages/catalog/releases/download/metrics/v0.0.0.post20261022/metrics-0.0.0.post20261022.tar.gz
```

Use the `weectl` from your WeeWX environment. Add
`--config /path/to/weewx.conf` when the station configuration is not found
automatically. Package installations may require `sudo`; activate the WeeWX
virtual environment for pip installations. Restart your WeeWX service after
configuration, for example `sudo systemctl restart weewx` on systemd.
The installer does not restart it for you.

The installer enables `MetricsReport` and installs `skins/Metrics`:

```ini
[StdReport]
    [[MetricsReport]]
        skin = Metrics
        enable = true
        HTML_ROOT = /var/www/html/weewx/metrics
```

Choose a web directory writable by WeeWX. Wait for an archive record or run
`weectl report run MetricsReport`. The report creates `metrics` and
`metrics_minimal`, not an HTML page. A minimal Prometheus scrape job is:

```yaml
scrape_configs:
  - job_name: weewx
    metrics_path: /weewx/metrics/metrics
    static_configs:
      - targets: ['weather.example.org:80']
```

Replace the host and URL path with your web server's actual address. Set
`scheme: https` if that server uses TLS. Data refreshes with WeeWX reports,
even if Prometheus scrapes more often. Values follow the report's units.

See the [exposition example](docs/metrics-example.md) for names and labels.
The templates expect the weather observations they reference; use the minimal
output or adapt templates when your station lacks those fields. Check the
rendered text, especially custom station labels, with your Prometheus parser.

## Upgrade and remove

Back up modified templates, then install a newer fixed catalog archive and
restart WeeWX. The upstream build script expects `skins/Metrics`; raw source
builds need to resolve the source directory's different case.
Remove with `weectl extension uninstall Metrics` and restart WeeWX.

## License and origin

The project by Thomas Kent, Alex Burnett and Sirius Insight derives from
[weewx-json](https://github.com/teeks99/weewx-json) and retains [GPLv3](LICENSE).
