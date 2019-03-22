# Partial success

This exercise was an almost-win. The gce Ansible module doesn't seem to support the creation of instances from templates. Without being able to use templates we can't easily create the managed groups based on those templates.

I tried to hack my way around it, and almost got there, but ran into a problem with adding pre-existing instances to a managed group.

For completeness, here's the partially successful playbook:

```
---
- name: create cloud resources
  hosts: localhost
  connection: local
  become: no
  gather_facts: no
  vars_files:
    - /home/jason/qwiklabs-gcp-essentials-ansible/vars.yml
  tasks:
    - name: Launch instances
      gce:
        service_account_email: "{{ service_account_email }}"
        credentials_file: "{{ credentials_file }}"
        project_id: "{{ project_id }}"
        name: nginx
        num_instances: 2
        machine_type: "{{ machine_type }}"
        image: "{{ image }}"
        zone: "{{ zone }}"
      register: gce

    - name: open port 80 in the firewall
      gce_net:
        service_account_email: "{{ service_account_email }}"
        credentials_file: "{{ credentials_file }}"
        project_id: "{{ project_id }}"
        name: default
        fwname: "http"
        allowed: tcp:80
        state: "present"

    - name: wait for ssh
      wait_for:
        delay: 1
        host: "{{ item.public_ip }}"
        port: 22
        state: started
        timeout: 120
      with_items: "{{ gce.instance_data }}"

    - name: make host group
      add_host:
        hostname: "{{ item.public_ip }}"
        groupname: gcelab
      with_items: "{{ gce.instance_data }}"

    - debug: var=groups
    - debug: var=gce

- name: configure instance OSes
  hosts: gcelab
  become: yes
  remote_user: jason
  gather_facts: yes
  vars_files:
    - /home/jason/qwiklabs-gcp-essentials-ansible/vars.yml
  tasks:
    - name: install nginx
      apt:
        update_cache: yes
        name: nginx
        state: present
    - name: start the service
      service:
        name: nginx
        state: started
    - name: fix index
      replace:
        path: /var/www/html/index.nginx-debian.html
        regexp: nginx
        replace: "Google Cloud Platform - '{{ ansible_hostname }}'"

- name: set up target pools
  hosts: localhost
  connection: local
  become: no
  gather_facts: no
  tasks:
    - name: create target pool
      shell: gcloud compute target-pools create nginx-pool
      args:
        creates: nginx-pool

    - name: add instances to pool
      shell: "gcloud compute target-pools add-instances nginx-pool --instances {{ item.name }}"
      args:
        creates: "target-pools-{{ item.name }}"
      with_items: "{{ gce.instance_data }}"

    - name: create the managed instance group
      shell: "gcloud compute instance-groups managed create nginx-group --size=2 --target-pool=nginx-pool --zone={{ zone }}"
      args:
        creates: nginx-group

    # no way to add instances to the managed instance-group

    - name: create l3 load balancer
      shell: "gcloud compute forwarding-rules create nginx-lb --region us-central1 --ports=80 --target-pool nginx-pool"
      args:
        creates: nginx-lb

    - name: create health check
      shell: gcloud compute http-health-checks create http-basic-check
      args:
        creates: http-bacic-check

    - name: define http service and map port
      shell: gcloud compute instance-groups managed set-named-ports nginx-group --named-ports http:80
      args:
        creates: http-service

    - name: create backend service
      shell: gcloud compute backend-services create nginx-backend --protocol HTTP --http-health-checks http-basic-check --global
      args:
        creates: nginx-backend

    - name: add backend
      shell: "gcloud compute backend-services add-backend nginx-backend --instance-group nginx-group --instance-group-zone {{ zone }} --global"
      args:
        creates: nginx-backend-add

    - name: create url map
      shell: gcloud compute url-maps create web-map --default-service nginx-backend
      args:
        creates: web-map

    - name: create target http proxy
      shell: gcloud compute target-http-proxies create http-lb-proxy --url-map web-map
      args:
        creates: http-proxy

    - name: create global forwarding rule
      shell: gcloud compute forwarding-rules create http-content-rule --global --target-http-proxy http-lb-proxy --ports 80
      args:
        creates: forwarding-rule

# left off here
```
