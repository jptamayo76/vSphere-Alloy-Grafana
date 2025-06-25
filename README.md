# vSphere-Alloy-Grafana

## Grafana Dashboard for VMware vCenter Server version 8.0

![image](https://github.com/user-attachments/assets/ce5da60a-4a84-4591-a2d5-4a878ec71971)

![image](https://github.com/user-attachments/assets/1c7d08e6-7a41-4266-a685-8f811896345f)

![image](https://github.com/user-attachments/assets/465c1162-da49-40f3-9554-a4f473277ba0)


## Installation

### 1. [Import Dashboard](#importing-the-dashboard)
  
- Log in to Grafana.

- Go to **Home > Dashboards > New > Import**.

- Select the ``VMware-vSphere-Dashboard.json`` file.


### 2. Select the Data Source

- Confirm that Grafana requests the **Prometheus datasource** as input.

- Select a **Prometheus** data source that points to the Prometheus server.

- If it doesn't exist, [create one](#creating-a-prometheus-data-source-that-points-to-the-prometheus-server):

  - **Type:** ``Prometheus``

  - **Prometheus server URL:** http://prometheus.server:9090

  - **Authentication:** ``No Authentication``

  - **TLS:** Configure as needed (optional).


### 3. Verify Dashboard Operation

- Verify that **all dashboards load data _correctly_**.

- Check for **data source or PromQL query errors**.

- Check that the **response times are _reasonable_**.


### 4. Save and Make Available

- If everything works correctly, click **Save Dashboard**.

- Optional: Assign read-only permissions to the team users.


### 5. Additional Notes

- The data source is **dynamic**; you can change it later if you migrate the Prometheus server.

- The dashboard supports **Period** filter.

- Development and testing were carried out on the following platform:
  | Server                | Version      | 
  |-----------------------|--------------|
  | VMware vCenter Server | 8.0.3.00500  |
  | Prometheus            | 2.31.2+ds1   |
  | Grafana Alloy         | 1.9.1-1      |

---

## Configuring Grafana Alloy to Collect Metrics from vCenter Server

Install **Grafana Alloy** on a server.

Configure the ``/etc/alloy/config.alloy`` file as follows:

~~~~
// Config for Alloy.
// For a full configuration reference, see https://grafana.com/docs/alloy
//
// ###############################
// #### Metrics Configuration ####
// ###############################

otelcol.receiver.vcenter "integrations_vsphere" {
  endpoint = "https://vcenter8.example.com"
  username = "Administrator@vsphere.local"
  password = "abc123"

  collection_interval = "1m"

  tls {
    insecure = true
  }

  output {
    metrics = [otelcol.processor.batch.integrations_vsphere.input]
  }
}

otelcol.processor.batch "integrations_vsphere" {
  output {
    metrics = [otelcol.processor.transform.integrations_vsphere.input]
  }
}

otelcol.processor.transform "integrations_vsphere" {
    error_mode = "ignore"

    metric_statements {
        context = "resource"
        statements = [
            `set(attributes["job"], "integrations/vsphere") where attributes["job"] == nil`,
        ]
    }

    output {
        metrics = [otelcol.exporter.prometheus.integrations_vsphere.input]
    }
}

otelcol.exporter.prometheus "integrations_vsphere" {
  forward_to = [prometheus.remote_write.local.receiver]
  resource_to_telemetry_conversion = true
}

// Configure a prometheus.remote_write component to send metrics to a Prometheus server.
prometheus.remote_write "local" {
  endpoint {
    url = "http://10.20.0.254:9090/api/v1/write"
  }
}
~~~~

Using the **otelcol.receiver.vcenter** component requires enabling the Alloy stability level in **_experimental_ mode**.

Configure the ``/etc/default/alloy`` file as follows:

~~~~
CUSTOM_ARGS="--server.http.listen-addr=0.0.0.0:12345 --stability.level=experimental"
~~~~

Restart the Alloy service.

~~~~
systemctl restart alloy
~~~~

You can check the service status with the following commands:

~~~~
systemctl status alloy
journalctl -u alloy -f
~~~~


> [!CAUTION]
> I have used the ``Administrator@vsphere.local`` user as an **_example_**, but you should create a vmware vsphere **read-only user** to use for monitoring purposes.

---

## Creating a Prometheus data source that points to the Prometheus server

- Go to **Home > Connections > Data sources**.
- Press the **"Add new data source"** button
- Choose a data source type: **Time series databases > Prometheus**
- Enter the correct values ​​for each item

![image](https://github.com/user-attachments/assets/83daad24-114d-4f16-9d8c-14c74be825a6)

![image](https://github.com/user-attachments/assets/ad27d1ee-db1b-4d17-ae52-97a3d603e98c)

- Press the **"Save & test"** button

![image](https://github.com/user-attachments/assets/77684671-7b9e-45ce-84b6-ef0f85e2d05c)


- If the response message **"Successfully queried the Prometheus API."** is displayed, then we can already use the Dashboard.

---

## Importing the Dashboard

- Go to **Home > Dashboards**.
- Press the **New** button.
- In the context menu that appears, select **Import**.

  ![image](https://github.com/user-attachments/assets/ea01296c-bc86-4e07-a226-b3e74a54c6e2)

- In the **Import dashboard** page, click on the **Import dashboard JSON file** box.

  ![image](https://github.com/user-attachments/assets/12ef5888-a8c7-4307-a97b-c920145a1eb9)

- A new window will appear to select the file to upload.
- Find the ``VMware-vSphere-Dashboard.json`` file in the location where you downloaded it, select it, and press the **"Open"** button to begin uploading it to the Grafana server.
- Confirm that Grafana requests the Prometheus datasource as input.

  ![image](https://github.com/user-attachments/assets/2c1a9b7e-3b7c-4f8f-863a-66324ecf3d72)

- Select the appropriate datasource for connection to the Prometheus Server

  ![image](https://github.com/user-attachments/assets/cc16112a-205f-4e81-a644-c11c6a96394d)


- Once the correct datasource has been configured, press the **Import** button.

  ![image](https://github.com/user-attachments/assets/f4996486-a4eb-4bc6-b202-bed545edd42a)


- If everything went well, you should now be able to see all the [Dashboard panels](#grafana-dashboard-for-vmware-vcenter-server-version-80) with information about your VMware vSphere infrastructure.
- Enjoy!


---
