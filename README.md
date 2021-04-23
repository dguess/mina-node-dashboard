# Mina Node Dashboard (Prometheus & Grafana)

This is a guide on how to set up a Mina Node Dashboard using Prometheus and Grafana. It uses Ubuntu 18.04 running on a t3.micro AWS EC2 instance.

## Step 1: Set up the new server

First up you need to set up a new server to run your Mina node dashboard. You can set this up to run on your Mina Block Producer server but it is **NOT** recommended.

### Create new AWS EC2 instance 

Start by createing a new AWS EC2 instance (https://aws.amazon.com/ec2/):

 - **Amazon Machine Image (AMI):** Ubuntu Server 18.04 LTS (HVM), SSD Volume Type
 - **Instance Type:** t3.micro

In this example we will use a t3.micro instance type but it likely work well on a server with less specifications.

### Configure Security Groups

Add the following Custom TCP rules to allow inbound traffic to Grafana and Prometheus:

 - **Grafana**
    - Type: Custom TCP Rule
    - Port Range: 3000
    - Description: Grafana
  - **Prometheus**
    - Type: Custom TCP Rule
    - Port Range: 9090
    - Description: Prometheus


## Step 2: Installing Prometheus

Next up you need to install Prometheus on your new server. Prometheus will collect all the real time metrics from our Mina node and store them in a time series database.

Connect to your new AWS instance and follow the following guide to install Prometheus:

https://computingforgeeks.com/install-prometheus-server-on-debian-ubuntu-linux/)

If everything worked correctly you should now be able to see Prometheus running by going to http://[IP Address]:9090

(replace [IP Address] with the public IP address of your prometheus server)

![Prometheus Running Example](./screenshot-prometheus-example.png)

## Configure Prometheus to get Mina metrics

Ok now you have Prometheus installed we need to set it up to get the metrics from the Mina node.

### Allow Prometheus access to Mina metrics

The Mina daemon needs some extra flags to allow the metrics to be accessed. If you're using the '~/.mina-env' file for this you can do the following.

Open the mina-env file for editing:

```shell
nano ~/.mina-env
```

Add the metric-ports flag to the 'EXTRA_FLAGS`

```shell
--metrics-port 6060
```

### Open ports on Mina server

Next you need to open some ports on your Mina node to allow Prometheus to communicate.

Create the follwoing firewall rule on your Mina server and allow the inbound traffic from Prometheus:

 - **Prometheus**
    - Type: Custom TCP Rule
    - Port Range: 6060, 9100
    - Source: [IP Address]
    - Description: Prometheus

(replace [IP Address] with the public IP address of your prometheus server)

### Configure Prometheus to retrieve metrics from the Mina server

Now we need to configure Prometheus to retrieve our Mina metrics.

On the prometheus server open the config file:

```shell
sudo nano /etc/prometheus/prometheus.yml
```

Update 'job_name' to `mina` and replace 'targets' with the IP address of your mina server:

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'mina'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['[IP_ADDRESS]:6060','[IP_ADDRESS]:9100']
```

(replace [IP Address] with the public IP address of your Mina server)

Now restart Prometheus:
```shell
sudo systemctl restart prometheus
```

If Prometheus is now working as expected we should be able to view the metrics collected from our Mina server.

Open the Prometheus web client again:
http://[IP_ADDRESS]:9090/

(replace IP_ADDRESS with the IP address of your Prometheus server)

Enter a valid mina metrics such as `Coda_Transition_frontier_max_blocklength_observed` and press execute.

If it's working as expected you should see the data for the metric selected similar to below:

![Prometheus Running Example](./screenshot-prometheus-example2.png)


## Step 4: Installing Grafana


