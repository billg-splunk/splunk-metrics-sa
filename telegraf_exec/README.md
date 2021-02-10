# Telegraf/exec

This example uses multipass and a simple script to demonstrate how the Telegraf/exec monitor can be used.

## Prepare Sample Application
* Create an ubuntu vm

```
multipass launch -n exec-example -d 4G -m 4G
multipass shell exec-example
```
* In Splunk IM, go to the Integrations page, select **SignalFx SmartAgent**, and copy the setup code and paste it in your vm. (You can instead copy the command below and replace **<ACCESS_TOKEN>** with yiour token.)

```
curl -sSL https://dl.signalfx.com/signalfx-agent.sh > /tmp/signalfx-agent.sh;
sudo sh /tmp/signalfx-agent.sh --realm us1 -- <ACCESS_TOKEN>
```
* Verify your agent is working with ```signalfx-agent status```
  * Verify Datapoints sent (last minute)
  * Note the number of Active and Configured Monitors (e.g. 10)
* Verify host is visible in Splunk IM - Infrastructure / Hosts (Smart Agent/collectd). It may take a few minutes to appear.

# Enable the Monitor
* Enable the monitor with the following. (This example creates a sample file but you can also edit the ```/etc/signalfx/agent.yaml``` file and get the same result):

```
sudo mkdir /etc/signalfx/monitors
sudo vi /etc/signalfx/monitors/exec.yaml
```
* Add the following to the file and save it

```
- type: telegraf/exec
  command: echo "myMeasurement,tag1=value1,tag2=value2 fieldKey=35"
  telegrafParser:
    dataFormat: influx
```

## Verify Results
* Several methods for verifying the result
  * ```signalfx-agent status```, and note the count of monitors increased
  * ```signalfx-agent status monitors | grep exec```, to see the new monitor and the metrics being sent
  * ```signalfx-agent tap-dps -metric '*myMeasurement*'```, to observe the datapoints for metrics containing 'jmx'
  * Login to Splunk Infrastructure
    * Navigate to **Metrics**, search for **myMeasurement**
    * Click on **myMeasurement** to see it in a chart