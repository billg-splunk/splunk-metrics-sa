# splunk-metrics-sa
Examples of adding metrics to Splunk Smart Agent

## Overview
The general process is the following:
* Find the customization you are looking to add
  * Main sources
    * [Smart Agent Documentation](https://docs.signalfx.com/en/latest/integrations/agent/monitors/index.html)
    * [Smart Agent Github List](https://github.com/signalfx/signalfx-agent/tree/master/docs/monitors)
  * If what you are looking for can't be found here you may be able to find other CollectD plugins or build your own collection
    * [CoillectD page](https://collectd.org/)
    * [CollectD Github](https://github.com/collectd/collectd)
    * [Python plugins directly](https://docs.signalfx.com/en/latest/integrations/agent/monitors/collectd-python.html)

## Examples
* [JMX](./jmx)