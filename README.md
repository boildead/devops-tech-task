#  Ansible Playbook for Grafana/Prometheus/AlertManager/Loki + Node-Exporter/Cadvisor Deployment in Docker containers
- Requires Ubuntu 20.04 server with sudo privileges for remote user
- Ansible 8.0 or higher
- Tested on Ubuntu 20.04 server

## Features
- Installs Grafana, Prometheus, AlertManager, Loki, Node-Exporter, Cadvisor in Docker containers
- Configures Grafana with Prometheus, Loki, AlertManager datasources
- Configures Grafana with basic dashboards
- Configures AlertManager with email notification
- Configures Prometheus with Node-Exporter, Cadvisor and Docker SD
- Configures Loki so Docker logs are collected and stored in Loki

When playbook is finished, you can access Grafana at `http://<your-server-address>:3000` with default admin user `admin` and password set in `GRAFANA_ADMIN_PASSWORD` or `secrets_file.enc`.
## Usage
- Set inventory file `production.ini` with your remote host
- Set env variables for GRAFANA_ADMIN_PASSWORD and ALERTMANAGER_EMAIL_PASSWORD
- Alternatively, you can use `ansible-vault` to set GRAFANA_ADMIN_PASSWORD (stored in `roles/common/files/secrets_file.enc`) using `ansible-playbook ... -e secrets_file.enc --ask-vault-pass site.yml`
- Set appropriate variables for
- Run `ansible-playbook -i production.ini site.yml`

## Variables
- Default variables for role stored in `roles/common/defaults/main.yml`