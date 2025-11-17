# Homelab Infrastructure

# Architektur

- **Firewall:** pfSense
- **VLANs:**
  - VLAN 10 – Server
  - VLAN 20 – Monitoring
  - VLAN 30 – IoT
- **Wichtige Systeme:**
  - Monitoring-VM (Docker, Grafana, InfluxDB, Node-RED)
  - IoT-VM (MQTT, Sensorik)
  - Automation-VM (Ansible, Git)

# Ziel

- Infrastruktur als Code mit **Ansible**
- Automatisierte Deployments über **CI/CD**
- Überwachung mit **Grafana** und **InfluxDB**
- Saubere Trennung von Netzwerksegmenten (VLANs)
