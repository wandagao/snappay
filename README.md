ansible-elk
===========
## What does it do?
   - Automated deployment of a full 6.8+ ELK or EFK stack (Elasticsearch, Logstash/Fluentd, Kibana)
     * `5.6` and `2.4` ELK versions are maintained as branches and `master` branch will be 6.x currently.
     * Uses Nginx as a reverse proxy for Kibana, or optionally Apache via `apache_reverse_proxy: true`
     * Generates SSL certificates for Filebeat or Logstash-forwarder
     * Adds either iptables or firewalld rules if firewall is active
     * Tunes Elasticsearch heapsize to half your memory, to a max of 32G
     * Deploys ELK clients using SSL and Filebeat for Logstash (Default)
     * Deploys rsyslog if Fluentd is chosen over Logstash, picks up
       the same set of OpenStack-related logs in /var/log/*
     * All service ports can be modified in ```install/group_vars/all.yml```
     * Optionally install [curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html)
     * Optionally install [Elastic X-Pack Suite](https://www.elastic.co/guide/en/x-pack/current/xpack-introduction.html)
     * This is also available on [Ansible Galaxy](https://galaxy.ansible.com/sadsfae/ansible-elk/)

## Requirements
   - RHEL7 or CentOS7 server/client with no modifications
   - ELK/EFK server with at least 8G of memory (you can try with less but 5.x series is quite demanding - try 2.4 series if you have scarce resources).
   - You may want to modify ```vm.swappiness``` as ELK/EFK is demanding and swapping kills the responsiveness.
     - I am leaving this up to your judgement.
```
echo "vm.swappiness=10" >> /etc/sysctl.conf
sysctl -p
```

## Notes
   - Current ELK version is 6.x but you can checkout the 5.6 or 2.4 branch if you want that series
   - Sets the nginx htpasswd to admin/admin initially
   - nginx ports default to 80/8080 for Kibana and SSL cert retrieval (configurable)
   - Uses OpenJDK for Java
   - It's fairly quick, takes around 3minutes on a test VM
   - Fluentd can be substituted for the default Logstash
     - Set ```logging_backend: fluentd``` in ```group_vars/all.yml```
   - Install curator by setting ```install_curator_tool: true``` in ```install/group_vars/all.yml```
   - Install [Elastic X-Pack Suite](https://www.elastic.co/guide/en/x-pack/current/xpack-introduction.html) for Elasticsearch, LogStash or Kibana via:
     - ```install_elasticsearch_xpack: true```
     - ```install_kibana_xpack: true```
     - ```install_logstash_xpack: true```
     - Note: Deploying X-Pack will wrap your ES with additional authentication and security, Kibana for example will have it's own credentials now - the default is username: ```elastic``` and password: ```changeme```

## ELK/EFK Server Instructions
   - Clone repo and setup your hosts file
```
git clone https://github.com/sadsfae/ansible-elk
cd ansible-elk
sed -i 's/host-01/elkserver/' hosts
sed -i 's/host-02/elkclient/' hosts
```
   - If you're using a non-root user for Ansible, e.g. AWS EC2 likes to use ec2-user then set the follow below, default is root.

```
ansible_system_user: ec2-user
```

   - Run the playbook
```
ansible-playbook -i hosts install/elk.yml
```
   - (see playbook messages)
   - Navigate to the ELK at http://host-01:80 (default, nginx) or http://host-01/kibana (apache)
   - Default login is:
      - username: ```admin```
      - password: ```admin```

