## Automated ELK Stack Deployment

The files in this repository were used to configure the network depicted below.

![TODO: Update the path with the name of your diagram](Images/diagram_filename.png)

The webservers listed in the diagram (web1/2/3) are the hosts being monitored by the ELK stack.

These files have been tested and used to generate a live ELK deployment on Azure. 

  - setupELK-playbook.yml
  - filebeat-playbook.yml
  - metricbeat-playbook.yml

This document contains the following details:
- Description of the Topology
- Access Policies
- ELK Configuration
  - Beats in Use
  - Machines Being Monitored
- How to Use the Ansible Build


### Description of the Topology

The main purpose of this network is to expose a load-balanced and monitored instance of DVWA, the D*mn Vulnerable Web Application.

Load balancing ensures that the application will be highly available, in addition to restricting access to the network.

- What aspect of security do load balancers protect? 
  - Load balancers are designed to distribute tasks over a set of resources, with the aim of making their overall processing more efficient. From the aspect of security, load balancers ensure "availability", but more importantly provide a single point of entry, which concerns itself with the mechanism of access control. Load balancers naturally protect against DDoS attacks (resiliency is created because of the distributed load), but they also control the traffic going to the backend pool of servers. Inbound NAT rules can be applied to Load Balancers which further hardens the backend pool from attack. 
- What is the advantage of a jump box?
  - The advantage of the jump box is that it is essentially controlling physical access to all other servers in the vnet. As a single point of entry (a door) into the virtual network environment, the attack surface is greatly reduced. By further creating network security rules for what traffic can reach the jump box, as well as ssh keys required for entry into the jump box, we are essentially creating a massive steal vault door with a cryptographic padlock.  

Integrating an ELK server allows users to easily monitor the vulnerable VMs for changes to the file and system metrics (health).
- What does Filebeat watch for?
  - Filebeat specifically logs information about the file system, including which files have changed and when.
- What does Metricbeat record?
  - Metricbeat helps you monitor your servers and the services they host by collecting metrics from the operating system and services - CPU usage, Disk I/O, Ram, etc. 

The configuration details of each machine may be found below.
_Note: Use the [Markdown Table Generator](http://www.tablesgenerator.com/markdown_tables) to add/remove values from the table_.

| Name       | Function                                      | IP Address | Operating System | Container ID |
| ---------- | --------------------------------------------- | ---------- | ---------------- | ------------ |
| Jump Box   | Gateway                                       | 10.0.0.4   | Linux Ubuntu 18  | 5562a8909131 |
| ELK Server | Hosting the ELK Monitoring stack              | 10.1.0.4   | Linux Ubuntu 20  | 6222c72bc2e3 |
| Web1       | WebServer (hosting DVWA) host to be monitored | 10.0.0.7   | Linux Ubuntu 20  | d19d42370dd4 |
| Web2       | WebServer (hosting DVWA) host to be monitored | 10.0.0.8   | Linux Ubuntu 20  | DVWA         |
| Web3       | WebServer (hosting DVWA) host to be monitored | 10.0.0.9   | Linux Ubuntu 20  | DVWA         |

### Access Policies

The machines on the internal network are not exposed to the public Internet. 

Only the jump box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- Add whitelisted IP address: <73.48.X.X> Which is my personal laptop public address (which is why I didn't include the entire IP!)

Machines within the network can only be accessed by the load balancer or the jump box.
- Which machine did you allow to access your ELK VM? The ELK VM is granted access from my personal laptop public address. What was its IP address? <73.48.X.X - not including the entire IP for security reasons.> 

A summary of the access policies in place can be found in the table below.

| Name          | Publicly Accessible | Allowed IP Addresses | Protocol/Port                                     |
| ------------- | ------------------- | -------------------- | ------------------------------------------------- |
| Jump Box      | Yes                 | 73.48.X.X            | SSH/22 \| HTTP/80                                 |
| ELK Server    | Yes                 | 73.48.X.X            | HTTP/80 \| ELK Stack Ports (5601,9300,9200, 5044) |
| Load Balancer | Yes                 | 73.48.X.X            | HTTP/80                                           |

### Elk Configuration

Ansible was used to automate configuration of the ELK machine. No configuration was performed manually, which is advantageous because...
- What is the main advantage of automating configuration with Ansible?
  - Automating the configuration of machines through Ansible makes systems easy to replicate, update, and control  the configuration. Many containers can be updated at one time which reduces administration time, errors, and misconfigurations. 

The playbook implements the following tasks:

- In 3-5 bullets, explain the steps of the ELK installation play. E.g., install Docker; download image; etc._
  - Step 1: Install Docker 
    - name: docker.io
          apt:
            force_apt_get: yes
            update_cache: yes
            name: docker.io
            state: present
  - Step 2: Install Python 
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
  - Step 3: Install Docker Python module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
  - Step 4: Download and install the ELK container and configure the ports
    - name: download and launch a docker web container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
         - 5601:5601 
         - 9200:9200 
         - 5044:5044

The following screenshot displays the result of running `docker ps` after successfully configuring the ELK instance.

<img src="C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606200048025.png" alt="image-20210606200048025" style="zoom:80%;" />

### Target Machines & Beats
This ELK server is configured to monitor the following machines:
- List the IP addresses of the machines you are monitoring
  - 10.0.0.7
  - 10.0.0.8
  - 10.0.0.9

We have installed the following Beats on these machines:
- Specify which Beats you successfully installed
  - Filebeat
  - Metricbeat

These Beats allow us to collect the following information from each machine:
- In 1-2 sentences, explain what kind of data each beat collects, and provide 1 example of what you expect to see. E.g., `Winlogbeat` collects Windows logs, which we use to track user logon events, etc.

  - Filebeat specifically logs information about the file system, including which files have changed and when.

    For example you can see the harvester is monitoring the auth.log:

     2021-06-07T03:10:10.152Z#011INFO#011log/harvester.go:251#011Harvester started for file: /var/log/auth.log

    This would tell us when a user gained authorization to the system - for example when a user logged in through SSH:

    <img src="C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606201418502.png" alt="image-20210606201418502" style="zoom:80%;" />

  - Metricbeat helps you monitor your servers and the services they host by collecting metrics from the operating system and services - CPU usage, Disk I/O, Ram, etc. 

  - For example you can see host O/S information (_source) by filtering hostname to see info specific to that host:

    <img src="C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606202140262.png" alt="image-20210606202140262" style="zoom:80%;" />

    And you can view what the overall container statistics for all containers across all hosts or just one of them as in the example below: (I have filtered for Web1)

    <img src="C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606202501610.png" alt="image-20210606202501610" style="zoom:80%;" />

### Using the Playbook
In order to use the playbook, you will need to have an Ansible control node already configured. Assuming you have such a control node provisioned: 

SSH into the control node and follow the steps below:
- Copy the playbooks to the Ansible control node. (jumpbox ansible container /etc/ansible/setupELK-playbook.yml)

- Update the ansible config file to include the name of the remote user:

  <img src="C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606204822562.png" alt="image-20210606204822562"  />

  Update the Ansible hosts file to indicate what hosts the playbooks should run on. In this case we have 2 sections - one for the Webservers running DVWA (which the ELK stack is monitoring) and one for the Elk server (where ELK stack should be installed).

  ​														 ![image-20210606211156484](C:\Users\Elena\AppData\Roaming\Typora\typora-user-images\image-20210606211156484.png)

- Run the playbook, and navigate to http://localhost:5601 to check that the installation worked as expected.

Answer the following questions to fill in the blanks:
- Which file is the playbook? Assuming we are talking about the ELK service playbook: **setupELK-playbook.yml**
-  Where do you copy it? I didn't copy it, I created a file using the touch command and then edited the file with nano and copied in the content needed for the playbook. That file should be here: /etc/ansible.
- Which file do you update to make Ansible run the playbook on a specific machine? **hosts file**
- How do I specify which machine to install the ELK server on versus which to install Filebeat on? **The location to install filebeat is configured with the filebeat-config.yml file. The ansible hosts file specifics where to install the ELK service.** 
- Which URL do you navigate to in order to check that the ELK server is running? The easiest way to know if the ELK stack is running is to navigate to the Kibana site:  **http://localhost:5601** (replace localhost with the ELK server IP)

_As a **Bonus**, provide the specific commands the user will need to run to download the playbook, update the files, etc._