# JMX

This example uses multipass and a sample application from Open-Telemetry. The example app's original intent is about manual tracing with Open Telemetry; we are only concerned with getting the JMX metrics.

These steps can be applied to other Java applications.

Things to keep in mind:
* The example assumes the agent is running on the same host as the application
* If it is running remotely (i.e. with Kubernetes, or with a remote agent) you will need to adjust settings to accomodate this

## Prepare Sample Application
* Create an ubuntu vm

```
multipass launch -n jmx-example -d 4G -m 4G
multipass shell jmx-example
```
* Install a Java Runtime Envrionment (JRE)

```
sudo apt install -y default-jre
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
* Install the sample application

```
cd ~
git clone https://github.com/open-telemetry/opentelemetry-java.git
cd open-telemetry/examples/http
../gradlew shadowJar
java -cp ./build/libs/opentelemetry-examples-http-0.1.0-SNAPSHOT-all.jar io.opentelemetry.example.http.HttpServer
```
* You can stop the server with ```<Control>-c``` once you have confirmed the application is running successfully 

## Enable JMX
* Per the instructions we need to enable JMX on the application if it isn't already.
* In our example app we can do that by running the application with the following command:

```
java -cp ./build/libs/opentelemetry-examples-http-0.1.0-SNAPSHOT-all.jar \
  -Dcom.sun.management.jmxremote.port=5000 \
  -Dcom.sun.management.jmxremote.authenticate=false \
  -Dcom.sun.management.jmxremote.ssl=false \
  io.opentelemetry.example.http.HttpServer
```
* You can run it in the background by appending **&** at the end of the line, e.g.

```
java -cp ./build/libs/opentelemetry-examples-http-0.1.0-SNAPSHOT-all.jar \
  -Dcom.sun.management.jmxremote.port=5000 \
  -Dcom.sun.management.jmxremote.authenticate=false \
  -Dcom.sun.management.jmxremote.ssl=false \
  io.opentelemetry.example.http.HttpServer &
```

## Enable the Monitor
* Enable the monitor with the following. (This example creates a sample file but you can also edit the ```/etc/signalfx/agent.yaml``` file and get the same result):

```
sudo mkdir /etc/signalfx/monitors
sudo vi /etc/signalfx/monitors/jmx.yaml
```
* Add the following to the file and save it

```
- type: collectd/genericjmx
  host: localhost
  port: 5000
```

## Verify Results
* Several methods for verifying the result
  * ```signalfx-agent status```, and note the count of monitors increased
  * ```signalfx-agent status monitors | grep jmx```, to see the new monitor and the metrics being sent
  * ```signalfx-agent tap-dps -metric '*jmx*'```, to observe the datapoints for metrics containing 'jmx'
  * Login to Splunk Infrastructure
    * Navigate to **Metrics**, search for **jmx**, and filter on **GenericJMX**
    * Navigate to **Dashboards**, search for **jmx**, and click on the **Generic java stats** under **JMX (collectd)

---

# Other Notes

## Todo
* Add pictures showing examples, as-needed
* Identify which metrics are necessary for the dashboard to be available
* Add information on remote collection, MBeans, etc.