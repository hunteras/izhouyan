---
title: "Elasticsearch with docker and ansible"
date: "2018-04-22T23:27:02+08:00"
lastmod: "2018-04-22T23:27:02+08:00"
draft: false
tags: ["izhouyan", "elasticsearch", "ansible", "devops", "docker"]
categories: ["elasticsearch", "ansible", "devops", "docker"]
author: "izhouyan"

---


Let's set up an elasticseach with kibana using docker and ansible.


Find or set up a machine with docker installed, we use localhost.

# Build docker image

```yaml
- hosts: localhost
  connection: local
  gather_facts: no

  tasks:
    - name: Working dir
      command: "pwd"
      register: work_dir

    - name: Show work dir
      debug:
        var: work_dir.stdout

    - name: build ubuntu images
      command: docker build -t ubuntu-aliyun:16.04 .
      args:
        chdir: "{{ work_dir.stdout }}/images/ubuntu-aliyun-16.04"

    - name: build jdk images
      command: docker build -t jdk:1.8 .
      args:
        chdir: "{{ work_dir.stdout }}/images/jdk-1.8"
        
    - name: build elastic images
      command: docker build -t elastic:5.4.3 .
      args:
        chdir: "{{ work_dir.stdout }}/images/elastic"

    - name: build kibana images
      command: docker build -t kibana:5.4.3 .
      args:
        chdir: "{{ work_dir.stdout }}/images/kibana"
    
```

# Deploy docker containers

```yaml
- hosts: localhost
  connection: local
  gather_facts: no

  environment:
    PYTHONPATH: /usr/local/lib/python2.7/site-packages
    
  tasks:
  - name: Working dir
    command: "pwd"
    register: work_dir

  - name: Create a network with options
    docker_network:
      name: elk-net
      ipam_options:
        subnet: '192.168.0.0/24'
        gateway: 192.168.0.1
      
  - name: restart elasticsearch container
    docker_container:
      name: elk-elastic
      image: elastic:5.4.3
      state: started
      restart: yes
      restart_policy: always
      user: elasticsearch
      command: /elasticsearch/bin/elasticsearch
      networks:
        - name: elk-net
          ipv4_address: "192.168.0.2"
      volumes:
        - "{{ work_dir.stdout }}/config/elastic/elasticsearch.yml:/elasticsearch/config/elasticsearch.yml"
      ports:
        - "9200:9200"
        - "9300:9300"        

  - name: restart kibana container
    docker_container:
      name: elk-kibana
      image: kibana:5.4.3
      state: started
      restart: yes
      restart_policy: always
      user: kibana
      command: /kibana/bin/kibana
      networks:
        - name: elk-net
          ipv4_address: "192.168.0.3"
      volumes:
        - "{{ work_dir.stdout }}/config/kibana/kibana.yml:/kibana/config/kibana.yml"
      ports:
        - "5601:5601"

```

We create a docker bridge net first, then set containers' ip, so containers can talk to each other.


That's it, simple and can run multiple es node contianer on different machines, just mod the ansible scripts.
