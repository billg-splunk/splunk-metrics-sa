# splunk-metrics-sa
Examples of adding metrics to Splunk Smart Agent

## Overview
The general process is the following:
* Find the customization you are looking to add
  * Main sources
    * [Smart Agent Documentation](https://docs.signalfx.com/en/latest/integrations/agent/monitors/index.html)
    * [Smart Agent Github List](https://github.com/signalfx/signalfx-agent/tree/master/docs/monitors)
  * If what you are looking for can't be found here you may be able to find other CollectD plugins or build your own collection
    * [coillectd page](https://collectd.org/)
    * [collectd Github](https://github.com/collectd/collectd)
    * [Python plugins directly](https://docs.signalfx.com/en/latest/integrations/agent/monitors/collectd-python.html)

## Examples
* [cassandra](./cassandra)
* [etcd](./etcd)
* [JMX](./jmx)
* [Telegraf/exec](./telegraf_exec)

## Some tips and tricks
Below are some common commands and tips you can use in modifying configurations and monitoring logs. They are specific to Linux/Ubuntu but could easily be adapted for other operating systems.

* Restarting the service
  * Many changes don't require restarting the agent but some (like log level) do
  * ```sudo service signalfx-agent restart```
* Logging datapoints
  * To view the datapoints
    * ```signalfx-agent tap-dps```
  * To send datapoints to the log
    * Change log level in ```/etc/signalfx/agent.yaml``` to **Debug**
    * Add the following to the ```writer``` section of ```/etc/signalfx/agent.yaml```
      * ```logDatapoints: true```
    * Restart the agent
    * Tail the log
      * ```journalctl -f -u signalfx-agent```
* To view monitors
  * Long output
    * ```signalfx-agent status monitors```
  * Short output
    * ```signalfx-agent status monitors | egrep '^[0-9]+\.'```
  * To get details about a specific monitor (e.g. etcd)
    * ```signalfx-agent status monitors | sed -n '/\. etcd/,/^$/p'```