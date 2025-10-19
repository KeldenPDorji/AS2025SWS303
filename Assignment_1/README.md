# **Assignment 1 – ELK & Filebeat Setup (Replayable Commands)**

**Environment:** Ubuntu VM on ARM Mac (Apple Silicon)

---

## **1. Update system & install curl**

```bash
sudo apt update
sudo apt install curl -y
```

*Screenshot idea:* `apt update` output showing successful updates.

---

## **2. Add Elastic GPG key and APT repository**

```bash
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic-archive-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
```

*Screenshot idea:* `sudo apt update` showing Elastic repo recognized.

---

## **3. Install Elasticsearch**

```bash
sudo apt install elasticsearch -y
```

*Screenshot idea:* Installation finished successfully.

---

## **4. Backup original configuration**

```bash
sudo cp /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.bak
```

---

## **5. Configure Elasticsearch (`elasticsearch.yml`)**

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

**Paste minimal working config:**

```yaml
cluster.name: elasticsearch
node.name: ubuntu
cluster.initial_master_nodes: ["ubuntu"]

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

http.host: 0.0.0.0
http.port: 9200

xpack.security.enabled: true
xpack.security.enrollment.enabled: true

xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.keystore.path: certs/http.p12

xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/transport.p12
xpack.security.transport.ssl.truststore.path: certs/transport.p12
```

*Screenshot idea:* Open file in nano showing the pasted configuration.

---

## **6. Start and enable Elasticsearch**

```bash
sudo systemctl restart elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl status elasticsearch
```

*Screenshot idea:* `systemctl status` output showing **active (running)**.

---

## **7. Reset the `elastic` user password**

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

* Copy the new password (example: `ZF6rks-myHeOrbCgrSOM`)

*Screenshot idea:* Terminal showing “Password successfully reset” message.

---

## **8. Test Elasticsearch connectivity**

```bash
curl -u elastic:ZF6rks-myHeOrbCgrSOM https://10.2.25.101:9200 --insecure
```

*Screenshot idea:* JSON output showing cluster info.

---

## **9. Install Filebeat**

```bash
sudo apt install filebeat -y
```

*Screenshot idea:* Filebeat installation output.

---

## **10. Configure Filebeat (`filebeat.yml`)**

```bash
sudo nano /etc/filebeat/filebeat.yml
```

**Paste working configuration:**

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/*.log

output.elasticsearch:
  hosts: ["10.2.25.101:9200"]
  username: "elastic"
  password: "ZF6rks-myHeOrbCgrSOM"

setup.kibana:
  host: "http://10.2.25.101:5601"

logging.level: info
logging.to_files: true
logging.files:
  path: /var/log/filebeat
  name: filebeat.log
  keepfiles: 7
  permissions: 0644

setup.dashboards.enabled: true

processors:
  - add_host_metadata: ~
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
```

*Screenshot idea:* Nano open with the configuration pasted.

---

## **11. Test Filebeat configuration**

```bash
sudo filebeat test config
# Should output: Config OK
```

*Screenshot idea:* Terminal showing `Config OK`.

---

## **12. Enable and start Filebeat**

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

*Screenshot idea:* `systemctl status filebeat` showing **active (running)**.

---

## **13. Verify Filebeat indexes in Elasticsearch**

```bash
curl -u elastic:ZF6rks-myHeOrbCgrSOM https://10.2.25.101:9200/_cat/indices?v --insecure
```

*Screenshot idea:* List of indexes showing `filebeat-*` index exists.

---

## **14. Test logs are searchable**

```bash
curl -u elastic:ZF6rks-myHeOrbCgrSOM https://10.2.25.101:9200/filebeat-*/_search?pretty --insecure
```

*Screenshot idea:* Sample JSON output of log documents.

---

### **Notes for submission**

* Backups: `elasticsearch.yml.bak`
* Security: TLS enabled (`--insecure` used for testing)
* IPs and credentials: `10.2.25.101`, `elastic`, `ZF6rks-myHeOrbCgrSOM`
* All commands are **replayable** on a fresh Ubuntu VM
