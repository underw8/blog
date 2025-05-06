# 🤖 How I monitor my Windows machines using Grafana

<figure><img src=".gitbook/assets/image (2) (1).png" alt=""><figcaption><p>Looks cool huh?</p></figcaption></figure>

## But why?

Well it's always a good idea to have something to trace back if either my machines is acting weird or broken down for no reason.

## What do you need to know before begin

Basic concept of Windows Services.

Know how to use Windows Terminal - Powershell or CMD.

Have an editor installed, either Notepad++ or Visual Studio Code (use Notepad? Fine, but beware of the file encoding and extension).

## Grafana Cloud and Grafana Agent

### Gafrana Cloud first

I'll cover this in another post, but yeah, I'm gonna use the Cloud version of Grafana since I don't trust any of my ISPs or Electrical Providers for my homeserver. The Free-tier of Grafana Cloud seems to be more than enough for me at the time.

### Granafa Agent for the rest

It's great, almost perfect replacement for Prometheus as we can now ship our logs and metrics directly to our cloud hosted Prometheus data source without the need of prometheus.

## Windows Logs via Grafana Agent

The default installation of Grafana Agent already included Windows Events logs, but if you need to collect other logs as well, simply add the follow configuration to the logs section of your `grafana-agent.yml`:

```
    - job_name: sd-log
      static_configs:
      - targets: [localhost]
        labels:
          job: applicationlog
          __path__: "C:\\Apps\\sd\\svc.log"
          instance: cdm-ws
```

Save the file and restart the Grafana Agent service via `service.msc`.

## Windows Metrics via OhmGraphite

Those forks at OhmGraphite - [https://github.com/nickbabcock/OhmGraphite](https://github.com/nickbabcock/OhmGraphite) did an amazing jobs of collecting not only CPU/MEM/... things that Grafana Agent can cover but also Power and GPU, and I really need these metrics.

Head over to their repo above and follow their installtion instruction (using their sample configuration, we'll edit them after). After having the exporter installed, let's export some metrics shall we?

Add the following to the `OhmGraphite.exe.config` - located in the same location of the exe file.

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <appSettings>
    <add key="type" value="prometheus" />
    <add key="prometheus_port" value="4445" />
    <add key="prometheus_host" value="localhost" />
    <add key="prometheus_path" value="metrics/" /> 
  </appSettings>
</configuration>
```

Save it and finally restart the OhmGraphite service via `services.msc`.

### Oh and don't forget the cool dashboard

Head over to [https://grafana.com/grafana/dashboards/11587](https://grafana.com/grafana/dashboards/11587) and copy the ID.

Go to your Grafana Cloud instance's dashboard - Click on New then Import, paste the ID and voila.
