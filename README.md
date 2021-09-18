## ELK-Stack-Project
**Automated ELK Stack Deployment**
 
The files in this repository were used to configure the network depicted below.

![vNet Diagram](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Diagrams/ELK-Project-V1.jpg)
 
These files have been tested and used to generate an automated ELK Stack Deployment on Azure. They can be used to either recreate the entire deployment figured below. Otherwise, select portions of the YAML files may be used to install only certain pieces of it, for example, Filebeat and Metricbeat.

  - install-elk.yml
  - filebeat-config.yml
  - filebeat-playbook.yml
  - metricbeat-config.yml
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

Load balancing ensures that the application will be highly available, in addition to restricting inbound access to the network.

What aspect of security do load balancers protect?
- According to [Azure security baseline for Azure Load Balancer](https://bit.ly/3AnSRPV), the load balancer's main purpose is to distribute web traffic across multiple servers. In our network, the load balancer was installed in front of the VM to 
++ protect Azure resources within virtual networks
+ monitor and log the configuration and traffic of virtual networks, subnets, and NICs
+ protect critical web applications
+ deny communications with known malicious IP addresses
+ record network packets
+ deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)
+ manage traffic to web applications
+ minimize complexity and administrative overhead of network security rules
+ maintain standard security configurations for network devices
+ document traffic configuration rules
+ use automated tools to monitor network resource configurations and detect changes

What is the advantage of a jump box?
- According to A Jump Box or a "Jump Server" is a gateway on a network used to access and manage devices in different security zones. A Jump Box acts as a security layer between networks and/or security zones and provides a controlled way to access them. Via the Jump Box, I make sure that access controls are in place to ensure that only authorized users (in this case, ourselves), will be able to connect to the network.

Integrating an Elastic Stack server allows users to easily monitor the vulnerable VMs for changes to their file systems and system metrics such as privilege escalation failures, SSH logins activity, CPU and memory usage, etc.

What does Filebeat watch for?
- According to

What does Metricbeat record?
- According to

The configuration details of each machine may be found below.
 
| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump Box | Gateway  | 10.0.0.4   | Linux            |
| Web-1        |webserver    | 10.0.0.5     | Linux            |
| Web-2        |webserver    | 10.0.0.6     | Linux            |
| ELKserver    |Kibana       | 10.1.0.4     | Linux            |
| Load Balancer|Load Balancer| 168.62.10.182| DVWA            |
 
In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into the following availability zones:


- Availability Zone 1: Web-1 + Web-2

- Availability Zone 2: ELK-Server

## ELK Server Configuration

The ELK Virtual Machine exposes an Elastic Stack instance. Docker is used to download and manage an ELK container.

Rather than configure Elastic Stack server manually, I opted to develop a reusable Ansible Playbook to accomplish the task. This playbook is duplicated below.

To use this playbook, one must log into the Jump Box, then issue the command: 

```bash
ansible-playbook install_elk.yml
```

This runs the install_elk.yml playbook on the elk host.

### Access Policies
 
The machines on the internal network are not exposed to the public Internet.
 
Only the _____ machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses:
- _TODO: Add whitelisted IP addresses_
 
Machines within the network can only be accessed by _____.
- _TODO: Which machine did you allow to access your ELK VM? What was its IP address?_
 
A summary of the access policies in place can be found in the table below.
 
| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump Box | Yes                 | 47.185.204.83        |
| ELK-SERVER      | No                  |  10.0.0.1-254        |
| DVWA 1   | No                  |  10.0.0.1-254        |
| DVWA 2   | No                  |  10.0.0.1-254        |

 
### Elk Configuration
 
Ansible was used to automate configuration of the ELK server. No configuration was performed manually, which is advantageous because Ansible can be used to easily configure new machines, update programs and configurations on hundreds of servers at once, and the best part is that the process is the same whether you're managing one machine or dozens and even hundreds.

The playbook implements the following tasks:

```yaml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: Web_1
  become: true
  tasks:
```

- In the above play, representing the header of the YAML file, I defined the title of my playbook based on the playbook's main goal by setting the keyword 'name:' to: "Configure Elk VM with Docker". 

Next I defined the managed nodes to target, in this case I set the keyword 'hosts:' to "elk", making sure that the playbook is run only on the machines in the "elk" group. 
To edit groups and add/remove machines from a group, the following inventory file located in /etc/ansible is used (see image below).

![hosts file editing](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/hosts_file_web_servers_edit.png) 

Next I defined the user account for the SSH connection, i.e., Web_1, by setting the keyword 'remote_user:' to "Web_1".

Next I activated privilege escalation by setting the keyword 'become:' to "true". 

Following the keyword 'tasks:', the second play is defined below.

```yaml
     # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
```

In this play, the ansible package manager module is tasked with installing docker.io. The keyword 'update_cache:' is set to "yes" to download package information from all configured sources and their dependencies prior to installing docker, it is necessary to successfully install docker in this case. Next the keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
```

In this play, the ansible package manager module is tasked with installing  'pip3', a version of the 'pip installer' which is a standard package manager used to install and maintain packages for Python.
The keyword 'force_apt_get:' is set to "yes" to force usage of apt-get instead of aptitude. The keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```

In this play the pip installer is used to install docker and also verify afterwards that docker is installed ('state: present').

```yaml
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```

In this play, the ansible sysctl module configures the target virtual machine (i.e., the Elk server VM) to use more memory. On newer version of Elasticsearch, the max virtual memory areas is likely to be too low by default (ie., 65530) and will result in the following error: "elasticsearch | max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]", thus requiring the increase of vm.max_map_count to at least 262144 using the sysctl module (keyword 'value:' set to "262144"). The keyword 'state:' is set to "present" to verify that the change was applied. The sysctl command is used to modify Linux kernel variables at runtime, to apply the changes to the virtual memory variables, the new variables need to be reloaded so the keyword 'reload:' is set to "yes" (this is also necessary in case the VM has been restarted).

```yaml
      # Use docker_container module
    - name: download and launch a docker elk container
      docker_container:
        name: elk
        image: sebp/elk:761
        state: started
        restart_policy: always
        published_ports:
          - 5601:5601
          - 9200:9200
          - 5044:5044
```

In this play, the ansible docker_container module is used to download and launch our Elk container. The container is pulled from the docker hub repository. The keyword 'image:' is set with the value "sebp/elk:761", "sebp" is the creator of the container (i.e., Sebastien Pujadas). "elk" is the container and "761" is the version of the container. The keyword 'state:' is set to "started" to start the container upon creation. The keyword 'restart_policy:' is set to "always" and will ensure that the container restarts if you restart your web vm. Without it, you will have to restart your container when you restart the machine.
The keyword 'published_ports:' is set with the 3 ports that are used by our Elastic stack configuration, i.e., "5601" is the port used by Kibana, "9200" is the port used by Elasticsearch for requests by default and "5400" is the default port Logstash listens on for incoming Beats connections (we will go over the Beats we installed in the following section "Target Machines & Beats").

```yaml
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```

In this play, the ansible systemd module is used to start docker on boot, setting the keyword 'enabled:' to "yes".


The following screenshot displays the result of running `docker ps` after successfully configuring the Elastic Stack instance.

![Docker ps output](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/elk_docker_ps_output.png)
 
### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.5
- Web-2 (DVWA 2) | 10.0.0.6

I have installed the following Beats on these machines:

- Filebeat
- Metricbeat

These Beats allow us to collect the following information from each machine:

`Filebeat`: Filebeat detects changes to the filesystem. I use it to collect system logs and more specifically, I use it to detect SSH login attempts and failed sudo escalations.

Filebeat playbook I used below:

```yaml
---
- name: Install and Launch Filebeat
  hosts: webservers
  become: yes
  tasks:
    # Use command module
  - name: Download filebeat .deb file
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.4.0-amd64.deb
    # Use command module
  - name: Install filebeat .deb
    command: dpkg -i filebeat-7.4.0-amd64.deb
    # Use copy module
  - name: Drop in filebeat.yml
    copy:
      src: /etc/ansible/files/filebeat-config.yml
      dest: /etc/filebeat/filebeat.yml
    # Use command module
  - name: Enable and Configure System Module
    command: filebeat modules enable system
    # Use command module
  - name: Setup filebeat
    command: filebeat setup
    # Use command module
  - name: Start filebeat service
    command: service filebeat start
    # Use systemd module
  - name: Enable service filebeat on boot
    systemd:
      name: filebeat
      enabled: yes
```


`Metricbeat`: Metricbeat detects changes in system metrics, such as CPU usage and memory usage.

Metricbeat playbook I used below:

```yaml
---
- name: Install and Launch Metricbeat
  hosts: webservers
  become: true
  tasks:
    # Use command module
  - name: Download metricbeat
    command: curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.4.0-amd64.deb
    # Use command module
  - name: install metricbeat
    command: dpkg -i metricbeat-7.4.0-amd64.deb
    # Use copy module
  - name: drop in metricbeat config
    copy:
      src: /etc/ansible/files/metricbeat-config.yml
      dest: /etc/metricbeat/metricbeat.yml
    # Use command module
  - name: enable and configure docker module for metric beat
    command: metricbeat modules enable docker
    # Use command module
  - name: setup metric beat
    command: metricbeat setup
    # Use command module
  - name: start metric beat
    command: service metricbeat start
    # Use systemd module
  - name: Enable service metricbeat on boot
    systemd:
      name: metricbeat
      enabled: yes
```
 
### Using the Playbook
In order to use the playbooks, you will need to have an Ansible control node already configured (I use my Jump Box as the Ansible control node), copy the playbooks to the Ansible control node and run the playbooks on the appropriate targets. 

First, I SSH into the control node and follow the steps below:

- Copy the playbook files to the Ansible control node.
- Update the "hosts" file to include the groups of hosts representing the targeted servers to run the playbooks on.
- Run the playbooks, and navigate to the ELK server to check that the installation worked as expected.

So first I connect to my Jump Box using the following command to SSH into the box:

```bash
ssh azadmin@51.141.166.114
```


![SSH into ump box](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/SSH_into_Jump_Box.png)

Then I run the following command to start and launch my Ansible docker container (i.e., the Ansible Control Node):

```bash
sudo docker start hopeful_lalande && sudo docker attach hopeful_lalande 
```

Note: Your container will have a different name.

![Start and launch ansible container](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/start_launch_ansible_container.png)

Then, I copy the playbooks into the correct location. The easiest way to do that is to use Git and run the following commands in your terminal:

```bash
cd /etc/ansible

mkdir files

# Clone Repository + IaC Files

git clone https://github.com/yourusername/projectname.git

# Move Playbooks and hosts file Into `/etc/ansible`

cp projectname/playbooks/* .

cp projectname/files/* ./files
```

Now that all the files I need are copied into the correct location, I can update the list of web servers to run the playbooks on:

I need to edit the "hosts" file located in /etc/ansible using the following commands:

```bash
nano hosts
```


Then I will update the file with the IP of the web servers we want to install Filebeat & Metricbeat & ELK on. To create a group I need to use brackets "[]", give the group of server a name (i.e., "webservers" & "elk") followed by the IP addresses of the servers.

![hosts file web server edit](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/hosts_file_web_servers_edit.png)

Next, I run the playbooks.

First I run my ELK playbook to deploy my ELK server:

```bash
ansible-playbook install_elk.yml
```

Then I run the Filebeat and Metricbeat playbooks to install the agents on my web servers (Web-1, Web-2):

```bash
ansible-playbook install_filebeat.yml
```
```bash
ansible-playbook install_metricbeat.yml
```


To verify that my ELK server was successfully deployed, I SSH into my ELK server and run the following command:

```bash
curl http://localhost:5601/app/kibana
```


If the server was successfully installed and deployed I should see the following HTML code output in the terminal:

![confirm elk server running via localhost](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/confirm_ELK_server_running_localhost.png)

You can also use your web browser to confirm that the ELK server is up and running by opening a web browser page and entering the public ip address to access Kibana's web interface:

http://40.79.255.121:5601/app/kibana

If the server is up and functioning, you should be able to access the page below:

![confirm elk running via public ip](https://github.com/Sk3llington/Project1-UCLA-Cyber-Security/blob/main/Images/confirm_ELK_server_running_public_ip.png)

Next, I want to verify that `filebeat` and `metricbeat` are actually collecting the data they are supposed to and that my deployment is fully functioning.

To do so, I have implemented 3 tasks:


1. Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.


2. Generate a high amount of CPU usage on my web servers and verify that Kibana picks up this data.


3. Generate a high amount of web requests to my web servers and make sure that Kibana is picking them up.


* Generating a high amount of failed SSH login attempts:


To generate these attempts I intentionally tried to connect to my Web-1 web server from the Jump Box instead of connecting from my Ansible container in order to generate failed attempts (the server can't verify my private key outside of the container).

To do so I used the following short script to automate 1000 failed SSH login attempts:


```bash
for i in {1..1000}; do ssh Web_1@10.0.0.5; done
```

![ssh failed attempts](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/f927b7cdbd50c0d4b7830f1839658fcfeaf2a96d/Images/ssh_failed_attempts.png)


Next I check Kibana to see if the failed attempts were logged:


![filebeat failed ssh attempts](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/f927b7cdbd50c0d4b7830f1839658fcfeaf2a96d/Images/filebeat_failed_ssh_attempts.png)

I can see that all the failed attempts were detected and sent to Kibana.

Now Let's breakdown the syntax of my previous short script:

`for` begins the `for` loop.

`i in` creates a variable named `i` that will hold each number `in` our list.

`{1..1000}` creates a list of 1000 numbers, each of which will be given to our `i` variable.

`;` separates the portions of our `for` loop when written on one line.

`do` indicates the action taken by each loop.

`ssh sysadmin@10.0.0.5` is the command run by `do`.

`;` separates the portions of our for loop when it's written on one line.

`done` closes the `for` loop.

Now I can run the same short script command with a few modifications, to test that `filebeat` is logging all failed attempts on all web servers where `filebeat` was deployed.

I want to run a command that will attempt to SSH into multiple web servers at the same time and continue forever until I stop it:

```bash
while true; do for i in {5..7}; do ssh Web_1@10.0.0.$i; done
```

Now let's breakdown the syntax of my previous short script:


`while` begins the `while` loop.

`true` will always be equal to `true` so this loop will never stop, unless you force quit it.

`;` separates the portions of our `while` loop when it's written on one line.

`do` indicates the action taken by each loop.

`i in` creates a variable named `i` that will hold each number in our list.

`{5..7}` creates a list of numbers (5, 6 and 7), each of which will be given to our `i` variable.

`ssh sysadmin@10.0.0.$i` is the command run by `do`. It is passing in the `$i` variable so the `wget` command will be run on each server, i.e., 10.0.0.5, 10.0.0.6, 10.0.0.7 (Web-1, Web-2).


Next, I want to confirm that `metricbeat` is functioning. To do so I will run a linux stress test.


* Generating a high amount of CPU usage on my web servers (Web-1, Web-2) and confirming that Kibana is collecting the data.


1. From my Jump Box, I start my Ansible container with the following command:

```bash
sudo docker start hopeful_lalande && sudo docker attach hopeful_lalande
```

2. I SSH from my Ansible container to one of my web server.

```bash
ssh Web_1@10.0.0.5
```

3. I install the `stress` module with the following command:

```bash
sudo apt install stress
```

4. I run the service with the following command and let the stress test run for a few minutes:

```bash
sudo stress --cpu 1
```

Next, I compare 2 of my web servers to see the difference in CPU usage, confirming that `metricbeat` is capturing the increase in CPU usage due to our stress command:

![cpu stress test results](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/7393789af6e4858bb3db389ed5271e2b712c6579/Images/cpu_stress_test_result.png)


Another view of the CPU usage metrics Kibana collected:

![cpu stress test results graph](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/cpu_stress_test_result_graph.png)


* Generate a high amount of web requests to my web servers and make sure that Kibana is picking them up.

This time I want to generate a high amount of web requests directed to one of my web servers, I will use `wget` to launch a DoS attack.

1. I log into my Jump Box

2. I need to add a new firewall rule to allow my Jump Box (10.0.0.4) to connect to my web servers over HTTP on port 80. To do so, I add a new Inbound Security Rule to my RedTeam1 Network Security Group:

![jump to http to webservers](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/jumpbox_http_to_webservers.png)


3. I run the following command to download the file `index.html` from my Web-1 VM:

```bash
wget 10.0.0.5
```

Output of the command:

![index html download](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/index_html_download.png)


4. I confirm that the file has been downloaded with the `ls` command:


```bash
azadmin@Jump-Box-Provisioner:~$ ls 
index.html
```

5. Next, I run the `wget` command in a loop to generate a very high number of web requests, I will use the `while` loop:

```bash
while true; do wget 10.0.0.5; done
```

The result is that the `Load`, `Memory Usage` and `Network Traffic` were hit as seen below:

![load increase DoS](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/load_increase_DoS.png)

![memory usage](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/memory_usage.png)

![network traffic increase](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/network_traffic_increase.png)

After stopping the `wget` command, I can see that thousands of index.html files were created (as seen below).


![index html files](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/9bcdcb0cdda628a18aad96fd07d56585c2b7a0cc/Images/index_html_files.png)


 I can use the following command to clean that up:

```bash
rm *
```

Now if we use `ls` again, the directory is a lot cleaner:


![directory cleanup](https://github.com/Sk3llington/Project-1-UCLA-Cyber-Security/blob/b3cb4729f2d776119d25fea2dcb676c6a22197c1/Images/directory_cleanup.png)


I can also avoid the creation of the `index.html` file by adding the flag `-O` to my command so that I can specify a destination file where all the `index.html` files will be concatenated and written to.

Since I don't want to save the `index.html` files, I will not write them to any output file but instead send them directly to a directory that doesn't save anything, i.e., `/dev/null`. 

I use the following command to do that:


```bash
while true; do wget 10.0.0.5 -O /dev/null; done
```

Now, if I want to perform the `wget` DoS request on all my web servers, I can use the previous command I used to generate failed SSH login attempts on all my web servers, but this time I will tweak the command to send `wget` requests to all 3 web servers:

```bash
while true; do for i in {5..7}; do wget -O /dev/null 10.0.0.$i; done
```

Note that I need to press CTRL + C to stop the `wget` requests since I am using the `while` loop.


My Elastic Stack server is now functioning and correctly monitoring my load-balanced exposed DVWA web application.
