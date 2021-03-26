# jenkins

This example uses multipass and an jenkins install to demonstrate how the jenkins monitor can be used.

## Install the Smart Agent
* Create an ubuntu vm

```
multipass launch -n jenkins-example -d 4G -m 4G
multipass shell jenkins-example
```
* In Splunk IM, go to the Integrations page, select **SignalFx SmartAgent**, and copy the setup code and paste it in your vm. (You can instead copy the command below and replace **<REALM>** with your realm and **<ACCESS_TOKEN>** with yiour token.)

```
curl -sSL https://dl.signalfx.com/signalfx-agent.sh > /tmp/signalfx-agent.sh;
sudo sh /tmp/signalfx-agent.sh --realm <REALM> -- <ACCESS_TOKEN>
```
* Verify your agent is working with ```signalfx-agent status```
  * Verify Datapoints sent (last minute)
  * Note the number of Active and Configured Monitors (e.g. 10)
* Verify host is visible in Splunk IM - Infrastructure / Hosts (Smart Agent/collectd). It may take a few minutes to appear.

## Install jenkins
* Run the following to install jenkins
  * You can go [here](https://www.jenkins.io/doc/book/installing/linux/) for more details
  * The install below is for the long term support release

```
sudo apt -y update
sudo apt install -y openjdk-8-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt-get -y update
sudo apt-get install -y jenkins
```

## Post-install Setup Wizard
* Once Jenkins is installed you need to go through the post-install setup wizard
* Inside the vm run ```sudo cat /var/lib/jenkins/secrets/initialAdminPassword``` and grab the password
* In your base OS (not inside the multipass vm) run ```multipass list``` and grab the **IP Address**
* Open your browser and navigate to http://**IP Address**:8080
* Paste the password in and click **Next**
* Select **Install Suggested Plugins**
* For creating the first admin click **Skip**
* Click **Save and Finish** and **Start Using Jenkins**

## Enable and verify Metrics
*  Go to Manage Jenkins > Manage Plugins > Available > Search "Metrics API to Jenkins" to find the plugin, check it, and click **Install without restart**.
* Go to Manage Jenkins > Configure System > Metrics > ADD, then click **Generate**, copy the **Key**, and then **Save**
* Check that metrics are working by navigating in the browser to http://**IP Address**:8080/metrics/**Key**

## Create good and bad builds
* From the dashboard click **New Item**
* Call it **Good**, select **Freestyle project**, and click **OK**
* Click **Save**
* Click **Back to the Dashboard**
* Create another **New Item**
* Call it **Bad**, select **Freestyle project**, click **OK**
* This time go to the **Build** section, click to **Add build step** and enter ```sh 'false'```
* Click **Save**
* Click **Back to the Dashboard**
* You now have two builds -- a good one and a bad one -- that you can run for testing

## Enable the Monitor
* Enable the monitor with the following. (This example creates a sample file but you can also edit the ```/etc/signalfx/agent.yaml``` file and get the same result):

```
sudo mkdir /etc/signalfx/monitors
sudo vi /etc/signalfx/monitors/jenkins.yaml
```
* Add the following to the file and save it

```
- type: collectd/jenkins
  host: localhost
  port: 8080
  metricsKey: **Key you copied**
  enhancedMetrics: true
```

## Verify Results
* Several methods for verifying the result
  * ```signalfx-agent status```, and note the count of monitors increased
  * ```signalfx-agent status monitors | grep jenkins```, to see the new monitor and the metrics being sent
  * ```signalfx-agent tap-dps -metric '*jenkins*'```, to observe the datapoints for metrics containing 'jenkins'
  * Go to the dashboards
    * Look at the Jenkins dashboards
  * Go to Metric
    * Navigate to **Metrics**, search for **jenkins**