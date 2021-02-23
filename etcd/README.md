# etcd

This example uses multipass and an etcd install to demonstrate how the etcd monitor can be used.

## Install the Smart Agent
* Create an ubuntu vm

```
multipass launch -n etcd-example -d 4G -m 4G
multipass shell etcd-example
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
  * You can go [here](https://github.com/etcd-io/etcd/releases/) for a later release

```
ETCD_VER=v3.4.14

# choose either URL
GOOGLE_URL=https://storage.googleapis.com/etcd
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GOOGLE_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

/tmp/etcd-download-test/etcd --version
/tmp/etcd-download-test/etcdctl version
```
* Start the server

```
/tmp/etcd-download-test/etcd &
```
* Write and read  etcd

```
/tmp/etcd-download-test/etcdctl --endpoints=localhost:2379 put foo bar
/tmp/etcd-download-test/etcdctl --endpoints=localhost:2379 get foo
```


# Enable the Monitor
* Enable the monitor with the following. (This example creates a sample file but you can also edit the ```/etc/signalfx/agent.yaml``` file and get the same result):

```
sudo mkdir /etc/signalfx/monitors
sudo vi /etc/signalfx/monitors/etcd.yaml
```
* Add the following to the file and save it

```
- type: etcd
  host: localhost
  port: 2379
```

## Verify Results
* Several methods for verifying the result
  * ```signalfx-agent status```, and note the count of monitors increased
  * ```signalfx-agent status monitors | grep etcd```, to see the new monitor and the metrics being sent
  * ```signalfx-agent tap-dps -metric '*etcd*'```, to observe the datapoints for metrics containing 'etcd'
  * Login to Splunk Infrastructure
    * Navigate to **Metrics**, search for **etcd**

## Adding additional metrics
etcd has a lot of custom metrics. Here is an example of adding additional ones
* See documentation [here](https://docs.signalfx.com/en/latest/integrations/agent/monitors/etcd.html) for examples of additional monitors
* Edit /etc/signalfx/monitors/etcd.yaml and replace the contents with the following
  
```
- type: etcd
  host: localhost
  port: 2379
  extraMetrics: [etcd_cluster_version,etcd_server_go_version]
```
* You should see two new custom metrics, which you can see using ```signalfx-agent status monitors```, in the GUI, etc.