# cassandra

This example uses multipass and a cassandra install to demonstrate how the cassandra monitor can be used.

## Install the Smart Agent
* Create an ubuntu vm

```
multipass launch -n cassandra-example -d 4G -m 4G
multipass shell cassandra-example
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

## Install etcd
* Run the following to install etcd
  * You can go [here](https://cassandra.apache.org/download/) for a later release
  * Check there if you encounter issues as well

```
sudo apt install -y openjdk-8-jre-headless
echo "deb https://downloads.apache.org/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra
```
* Start the server

```
sudo service cassandra start
```

# Verify it is running

```
nodetool status
```

# Enable the Monitor
* Enable the monitor with the following. (This example creates a sample file but you can also edit the ```/etc/signalfx/agent.yaml``` file and get the same result):

```
sudo mkdir /etc/signalfx/monitors
sudo vi /etc/signalfx/monitors/cassandra.yaml
```
* Add the following to the file and save it

```
- type: collectd/cassandra
  host: localhost
  port: 7199
```

## Verify Results
* Several methods for verifying the result
  * ```signalfx-agent status```, and note the count of monitors increased
  * ```signalfx-agent status monitors | grep cassandra```, to see the new monitor and the metrics being sent
  * ```signalfx-agent tap-dps -metric '*cassandra*'```, to observe the datapoints for metrics containing 'etcd'
  * Login to Splunk Infrastructure
    * Navigate to **Metrics**, search for **cassandra**

