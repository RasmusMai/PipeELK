# Setting up

# :warning: Read this! :warning:
**I've made significant improvements to this repository since the initial submission. Instead of updating the main branch, I've put everything new in the [`improvements`](https://github.com/RasmusMai/PipeELK/tree/improvements) branch.**

---

Please switch to the `improvements` branch to see the latest changes and improvements. Thank you for your attention!

# Overview

This is a pure-Ansible realization of the given exercise. It is not intended to be used in production, but rather as a proof of concept. My approach focused on pure automation, rather than security or performance.

The services are currently deployed on bare VMs as DigitalOcean droplets. Initially, I debated the utility of Docker, but concluded that each ElasticSearch instance required its own VM host, rendering Docker unnecessary.

Due to time constraints, I could not secure the internal workings of the system in its current phase. Although I have used ElasticSearch and Kibana as an end-user, this was my first attempt at setting the stack up.

I opted for ElasticSearch 8.6 instead of the more widely used 7.6 to explore its differences. In retrospect, I would have chosen differently, as the increased complexity and lack of community support consumed more time than expected.

I intentionally avoided using managed services offered by cloud providers, such as load balancers or orchestrators, as they were outside the scope of the given exercise.

There is no traditional inventory file, as I opted to use the DigitalOcean API to dynamically create droplets. This is done through the `community.digitalocean` module. The same module allows me to build an in-memory inventory.

The `community.crypto` module is used to generate self-signed certificates for nginx.

# Backups

Though there are no actual backups implemented, I would have used a combination of the following:
- Snapshotting ElasticSearch cluster as per the official [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)
    - Ideally storing them on DigitalOcean's block storage
    - Daily incrementals, weekly fulls
- Other aspects of the environment currently store no data, so they would not require backups.
    - These can be entirely restored using configuration management.

# Set up
## Requirements

- Ansible
- Ansible dependencies with `ansible-galaxy install -r requirements.yaml`
- DigitalOcean API token

## Configuration

- Copy `secret.yaml.example` to `secret.yaml` and add the DigitalOcean API token
- Review `group_vars/all.yaml` and change the variables as needed

## Run

1. Run `ansible-playbook -i deploy.yaml`
2. Wait until complete
3. Visit the DNS name of the Kibana droplet

Which will create:
- 5 droplets
    - 3x ElasticSearch 8.6
    - 1x Kibana
    - 1x Logstash
- DNS A record for Kibana
- Filebeats on all hosts
- nginx for Kibana with a self-signed certificate
- Internal communication though not secured, is only done on the private network
- Some standard OS configuration
    - Fail2Ban
    - Unattended upgrades

## Destroy

- Run `ansible-playbook -i destroy.yaml`

Will destroy all droplets and the associated DNS record.

# Checklist
- [x] A cluster of 3 Elasticsearch nodes 
- [x] 1 Logstash node 
    - [x] Logstash listening on TCP port 5042 
    - [x] Logstash can write to Elasticsearch 
- [x] 1 Kibana node 
    - [x] Kibana listening on a public interface 
    - [x] Kibana can read from Elasticsearch 
Bonus tasks: 
- [x] Automate virtual machine and/or container creation and orchestration  Can use the **same** or some other automation framework
- [ ] Add monitoring service of your choice: Nagios, Zabbix, Grafana, NewRelic etc.  CPU usage and disk space monitoring of every server is enough for this task 
- [ ] Describe the backup approach for all components (free text)
- [ ] Use standard HTTP ports and set up a TLS for all services 
- [x] Self-signed certificates are okay 
