
<img align="center" width="5600" height="530" src="https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/splashscreen.jpg">

---

## ELK-Stack-Project
**Automated ELK Stack Deployment**
 
The files in this repository were used to configure the network depicted below.

![vNet Diagram](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Diagrams/ELK-Project-V1.jpg)
 
These files have been tested and used to generate an automated ELK Stack Deployment on Azure. They can be used to either recreate the entire deployment figured below. Otherwise, select portions of the YAML files may be used to install only certain pieces of it, for example, Filebeat and Metricbeat.

  - [install-elk.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/install-elk.yml)
  - [filebeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-config.yml)
  - [filebeat-playbook.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-playbook.yml)
  - [metricbeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-config.yml)
  - [metricbeat-playbook.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-playbook.yml)
 
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

> What aspect of security do load balancers protect?
- According to [Azure security baseline for Azure Load Balancer](https://bit.ly/3AnSRPV), the load balancer's main purpose is to distribute web traffic across multiple servers. In our network, the load balancer was installed in front of the VM to 
   - protect Azure resources within virtual networks.
   - monitor and log the configuration and traffic of virtual networks, subnets, and NICs.
   - protect critical web applications
   - deny communications with known malicious IP addresses
   - record network packets
   - deploy network-based intrusion detection/intrusion prevention systems (IDS/IPS)
   - manage traffic to web applications
   - minimize complexity and administrative overhead of network security rules
   - maintain standard security configurations for network devices
   - document traffic configuration rules
   - use automated tools to monitor network resource configurations and detect changes


> What is the advantage of a jump box?
- A Jump Box or a "Jump Server" is a gateway on a network used to access and manage devices in different security zones. A Jump Box acts as a "bridge" between two trusted networks zones and provides a controlled way to access them. We can block the public IP address associated with the VM. It helps to improve security also prevents all Azure VM’s to expose to the public.

Integrating an Elastic Stack server allows us to easily monitor the vulnerable VMs for changes to their file systems and system metrics such as privilege escalation failures, SSH logins activity, CPU and memory usage, etc.

> What does Filebeat watch for?
- Filebeat helps keep things simple by offering a lightweight way (low memory footprint) to forward and centralize logs, files and watches for changes.

> What does Metricbeat record?
- Metricbeat helps monitor servers by collecting metrics from the system and services running on the server so it records machine metrics and stats, such as uptime.

The configuration details of each machine may be found below.
 
| Name     | Function | IP Address | Operating System |
|----------|----------|------------|------------------|
| Jump-Box-Provisioner | Gateway  | 44.77.55.33 ; 10.0.0.4   | Linux            |
| Web-1        |webserver    | 10.0.0.5     | Linux            |
| Web-2        |webserver    | 10.0.0.6     | Linux            |
| ELKServer    |Kibana       | 104.45.159.216 ; 10.1.0.4     | Linux            |
| RedTeam-LB|Load Balancer| 40.122.215.16| DVWA            |
 
In addition to the above, Azure has provisioned a load balancer in front of all machines except for the jump box. The load balancer's targets are organized into availability zones: Web-1 + Web-2


### Access Policies
 
The machines on the internal network are not exposed to the public Internet.
 
Only the Jump Box machine can accept connections from the Internet. Access to this machine is only allowed from the following IP addresses: 47.185.204.83 Machines within the network can only be accessed by SSH from Jump Box.
 
A summary of the access policies in place can be found in the table below.
 
| Name     | Publicly Accessible | Allowed IP Addresses |
|----------|---------------------|----------------------|
| Jump-Box-Provisioner | Yes                 | 47.185.204.83        |
| ELKServer      | Yes                  |  47.185.204.83:5601        |
| DVWA 1   | No                  |  10.0.0.1-254        |
| DVWA 2   | No                  |  10.0.0.1-254        |


 
---


### ELK Configuration
 
Ansible was used to automate the configuration of the ELK server. No configuration was performed manually, which is advantageous because Ansible can be used to easily configure new machines, update programs, and configurations on hundreds of servers at once, and the best part is that the process is the same whether we're managing one machine or dozens and even hundreds.

> What is the main advantage of automating configuration with Ansible?
- Ansible is focusing on bringing a server to a certain state of operation.

<details>
<summary> <b> Click here to view ELK Configuration. </b> </summary>

---
 
We will configure an ELK server within virtual network. Specifically,
 
- Deployed a new VM on our virtual network.
- Created an Ansible play to install and configure an ELK instance.
- Restricted access to the new server.

#### Deployed a new VM on our virtual network. 
 
1. Create a new vNet located in the same resource group we have been using. 
- Make sure this vNet is located in a new region and not the same region as our other VM's, which region we select is not important as long as it's a different US region than our other resources, we can also leave the rest of the settings at default.
- In this example, that the IP Addressing has automatically created a new network space of 10.1.0.0/16. If our network is different (10.2.0.0 or 10.3.0.0) it is ok as long as we accept the default settings. Azure automatically creates a network that will work.

![Create vNet](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Create%20vNet.png)  

2. Create a Peer connection between our vNets. This will allow traffic to pass between our vNets and regions. This peer connection will make both a connection from our first vNet to our second vNet and a reverse connection from our second vNet back to our first vNet. This will allow traffic to pass in both directions.
- Navigate to `Virtual Network` in the Azure Portal.
- Select our new vNet to view it's details.
- Under `Settings` on the left side, select `Peerings`.
- Click the + Add button to create a new Peering.
- A unique name of the connection from our new vNet to our old vNet such as depicted example below.
- Choose our original RedTeam vNet in the dropdown labeled `Virtual Network`.
- Leave all other settings at their defaults.
 
![PeeringsELKtoRed](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/ELKtoRed.png) 
 
![PeeringsRedtoELK](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/RedtoELK.png)  

3. Create a new Ubuntu VM in our virtual network with the following configurations:
- The VM must have a public IP address.
- The VM must be added to the new region in which we created our new vNet. We want to make sure we select our new vNEt and allow a new basic Security Group to be created for this VM.
- The VM must use the same SSH keys as our WebserverVM's. This should be the ssh keys that were created on the Ansible container that's running on our jump box.
- After creating the new VM in Azure, verify that it works as expected by connecting via SSH from the Ansible container on our jump box VM.

   - ```bash
        ssh sysadmin@<jump-box-provisioner>
     ``` 
   - ```bash
        sudo docker container list -a
     ``` 
   - ```bash
        sudo docker start goofy_wright && sudo docker attach goofy_wright
     ``` 
 
![connect_on_newVM](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/connect_on_newVM.png)  
 
- Copy the SSH key from the Ansible container on our jump box:
   - RUN `cat id_rsa.pub` Configure a new VM using that SSH key.
 
![RSA](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/id_rsa.pub_on_newVM.png) 
 

#### Created an Ansible play to install and configure an ELK instance.

In this step, we have to:
- Add our new VM to the Ansible hosts file.
- Create a new Ansible playbook to use for our new ELK virtual machine.
- From our Ansible container, add the new VM to Ansible's hosts file.
   - RUN `nano /etc/ansible/hosts` and put our IP with `ansible_python_interpreter=/usr/bin/python3`

![hosts file editing](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/CatHosts.png)  

- In the below play, representing the header of the YAML file, I defined the title of my playbook based on the playbook's main goal by setting the keyword 'name:' to: "Configure Elk VM with Docker". Next, I defined the user account for the SSH connection, by setting the keyword 'remote_user:' to "sysadmin" then activated privilege escalation by setting the keyword 'become:' to "true". 
 
 The playbook implements the following tasks:

```yaml
---
- name: Configure Elk VM with Docker
  hosts: elk
  remote_user: sysadmin
  become: true
  tasks:
```
 
In this play, the ansible package manager module is tasked with installing docker.io. The keyword 'update_cache:' is set to "yes" to download package information from all configured sources and their dependencies prior to installing docker, it is necessary to successfully install docker in this case. Next the keyword 'state:' is set to "present" to verify that the package is installed.


```yaml
     # Use apt module
    - name: Install docker.io
      apt:
        update_cache: yes
        name: docker.io
        state: present
```

In this play, the ansible package manager module is tasked with installing  'pip3', a version of the 'pip installer' which is a standard package manager used to install and maintain packages for Python.
The keyword 'force_apt_get:' is set to "yes" to force usage of apt-get instead of aptitude. The keyword 'state:' is set to "present" to verify that the package is installed.

```yaml
      # Use apt module
    - name: Install pip3
      apt:
        force_apt_get: yes
        name: python3-pip
        state: present
```

In this play the pip installer is used to install docker and also verify afterwards that docker is installed ('state: present').

```yaml
      # Use pip module
    - name: Install Docker python module
      pip:
        name: docker
        state: present
```

In this play, the ansible sysctl module configures the target virtual machine (i.e., the Elk server VM) to use more memory. On newer version of Elasticsearch, the max virtual memory areas is likely to be too low by default (ie., 65530) and will result in the following error: "elasticsearch | max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]", thus requiring the increase of vm.max_map_count to at least 262144 using the sysctl module (keyword 'value:' set to "262144"). The keyword 'state:' is set to "present" to verify that the change was applied. The sysctl command is used to modify Linux kernel variables at runtime, to apply the changes to the virtual memory variables, the new variables need to be reloaded so the keyword 'reload:' is set to "yes" (this is also necessary in case the VM has been restarted).

```yaml
      # Use sysctl module
    - name: Use more memory
      sysctl:
        name: vm.max_map_count
        value: "262144"
        state: present
        reload: yes
```

In this play, the ansible docker_container module is used to download and launch our Elk container. The container is pulled from the docker hub repository. The keyword 'image:' is set with the value "sebp/elk:761", "sebp" is the creator of the container (i.e., Sebastien Pujadas). "elk" is the container and "761" is the version of the container. The keyword 'state:' is set to "started" to start the container upon creation. The keyword 'restart_policy:' is set to "always" and will ensure that the container restarts if we restart our web vm. Without it, we will have to restart our container when we restart the machine.
The keyword 'published_ports:' is set with the 3 ports that are used by our Elastic stack configuration, i.e., "5601" is the port used by Kibana, "9200" is the port used by Elasticsearch for requests by default and "5400" is the default port Logstash listens on for incoming Beats connections (we will go over the Beats we installed in the following section "Target Machines & Beats").

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

In this play, the ansible systemd module is used to start docker on boot, setting the keyword 'enabled:' to "yes".

```yaml
      # Use systemd module
    - name: Enable service docker on boot
      systemd:
        name: docker
        enabled: yes
```
![Install_elk_yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Install_elk_yml.png)

Now we can start launching and exposing the container by run

```bash
ansible-playbook install-elk.yml
```

The following screenshot displays the result of running `install-elk.yml`

![Docker ELKResult output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Install_elk_result.png)

SSH to our container: ```ssh sysadmin@10.1.0.4``` and RUN ```sudo docker ps```

The following screenshot displays the result of running `docker ps` after successfully configuring the Elastic Stack instance.

![Docker InstallELK output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/InstallELK.png)

Logging into the Elk server and manually launch the ELK container with: 

```bash
sudo docker start elk
```
then ```curl http://localhost:5601/app/kibana``` does return HTML.

The following screenshot displays the result of running `curl` after start ELK container

![Docker curl output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/CurlResult.png)

#### Restricted access to the new server.
	
This step is to restrict access to the ELK VM using Azure's network security groups (NSGs). We need to add public IP address to a whitelist, just as we did when clearing access to jump box.

Go to Network Security Group to config our host IP to Kibana as follow

![Docker InboundSecRules output](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Docker%20InboundSecRules%20output.png)

Then try to access web browser to http://<your.ELK-VM.External.IP>:5601/app/kibana 
 
![Access_Kibana](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Access_Kibana.png)

</details>

---

### Target Machines & Beats
This ELK server is configured to monitor the following machines:

- Web-1 (DVWA 1) | 10.0.0.5
- Web-2 (DVWA 2) | 10.0.0.6

I have installed the following Beats on these machines:

- Filebeat
- Metricbeat

<details>
<summary> <b> Click here to view Target Machines & Beats. </b> </summary>

---

	
These Beats allow us to collect the following information from each machine:

`Filebeat`: Filebeat detects changes to the filesystem. I use it to collect system logs and more specifically, I use it to detect SSH login attempts and failed sudo escalations.

We will create a [filebeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/filebeat-config.yml) and [metricbeat-config.yml](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Ansible/metricbeat-config.yml) configuration files, after which we will create the Ansible playbook files for both of them.

Once we have this file on our Ansible container, edit it as specified:
- The username is elastic and the password is changeme.
- Scroll to line #1106 and replace the IP address with the IP address of our ELK machine.
output.elasticsearch:
hosts: ["10.1.0.4:9200"]
username: "elastic"
password: "changeme"
- Scroll to line #1806 and replace the IP address with the IP address of our ELK machine.
	setup.kibana:
host: "10.1.0.4:5601"
- Save both files filebeat-config.yml and metricbeat-config.yml into `/etc/ansible/files/`

![files_FMconfig](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/files_FMconfig.png) 
 
 
Next, create a new playbook that installs Filebeat & Metricbeat, and then create a playbook file, `filebeat-playbook.yml` & `metricbeat-playbook.yml`

RUN `nano filebeat-playbook.yml` to enable the filebeat service on boot by Filebeat playbook template below:

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
      src: /etc/ansible/roles/install-filebeat/files/filebeat-config.yml
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

![Filebeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook.png) 
 
- RUN `ansible-playbook filebeat-playbook.yml`

![Filebeat_playbook_result](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_result.png)  

Verify that our playbook is completed by navigate back to the Filebeat installation page on the ELK server GUI

![Filebeat_playbook_verify](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_verify.png)
	
![Filebeat_playbook_verify1](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Filebeat_playbook_verify1.png)
		
	
`Metricbeat`: Metricbeat detects changes in system metrics, such as CPU usage and memory usage.

RUN `nano metricbeat-playbook.yml` to enable the metricbeat service on boot by Metricbeat playbook template below:

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
      src: /etc/ansible/roles/install-metricbeat/files/metricbeat-config.yml
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

![Metricbeat_playbook](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook.png)  
 
- RUN `ansible-playbook metricbeat-playbook.yml`

![Metricbeat_playbook_result](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook_result.png)  

Verify that this playbook is completed by navigate back to the Filebeat installation page on the ELK server GUI

![Metricbeat_playbook_verify](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/Metricbeat_playbook_verify.png)

 
</details>

---
 
### Using the Playbook

Next, I want to verify that `filebeat` and `metricbeat` are actually collecting the data they are supposed to and that my deployment is fully functioning.

To do so, I have implemented 3 tasks:

1. Generate a high amount of failed SSH login attempts and verify that Kibana is picking up this activity.
2. Generate a high amount of CPU usage on my web servers and verify that Kibana picks up this data.
3. Generate a high amount of web requests to my web servers and make sure that Kibana is picking them up.
	
<details>
<summary> <b> Click here to view Using the Playbook. </b> </summary>

---


#### Generating a high amount of failed SSH login attempts:

To generate these attempts I intentionally tried to connect to my Web-1 web server from the Jump Box instead of connecting from my Ansible container in order to generate failed attempts (the server can't verify my private key outside of the container). All ELK Stack scripts refer to [Elk_Stack_scripts.sh](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Linux/Elk_Stack_scripts.sh)

To do so I used the following short script to automate 1000 failed SSH login attempts: 


```bash
for i in {1..1000}; do ssh Web_1@10.0.0.5; done
```

![ssh failed attempts](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/ssh%20failed%20attempts.png)


Next We check Kibana to see if the failed attempts were logged:


![filebeat failed ssh attempts](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/filebeat%20failed%20ssh%20attempts.png)

I can see that all the failed attempts were detected and sent to Kibana.

- Now Let's breakdown the syntax of my previous short script:

   - `for` begins the `for` loop.

   - `i in` creates a variable named `i` that will hold each number `in` our list.

   - `{1..1000}` creates a list of 1000 numbers, each of which will be given to our `i` variable.

   - `;` separates the portions of our `for` loop when written on one line.

   - `do` indicates the action taken by each loop.

   - `ssh sysadmin@10.0.0.5` is the command run by `do`.

   - `;` separates the portions of our for loop when it's written on one line.

   - `done` closes the `for` loop.

- Now I can run the same short script command with a few modifications, to test that `filebeat` is logging all failed attempts on all web servers where `filebeat` was deployed.

I want to run a command that will attempt to SSH into multiple web servers at the same time and continue forever until I stop it:

```bash
while true; do for i in {5..6}; do ssh Web_1@10.0.0.$i; done
```

- Now let's breakdown the syntax of my previous short script:

   - `while` begins the `while` loop.

   - `true` will always be equal to `true` so this loop will never stop, unless we force quit it.

   - `;` separates the portions of our `while` loop when it's written on one line.

   - `do` indicates the action taken by each loop.

   - `i in` creates a variable named `i` that will hold each number in our list.

   - `{5..6}` creates a list of numbers (5 and 6), each of which will be given to our `i` variable.

   - `ssh sysadmin@10.0.0.$i` is the command run by `do`. It is passing in the `$i` variable so the `wget` command will be run on each server, i.e., 10.0.0.5, 10.0.0.6 (Web-1, Web-2).


Next, I want to confirm that `metricbeat` is functioning. To do so I will run a linux stress test.


#### Generating a high amount of CPU usage on my web servers (Web-1, Web-2) and confirming that Kibana is collecting the data.


1. From my Jump Box, I start my Ansible container with the following command:

```bash
sudo docker start goofy_wright && sudo docker attach goofy_wright
```

2. Then, SSH from my Ansible container to Web-1.

```bash
ssh sysadmin@10.0.0.5
```

3. Install the `stress` module with the following command:

```bash
sudo apt install stress
```

4. Run the service with the following command and let the stress test run for a few minutes:

```bash
sudo stress --cpu 1
```

   - _Note: The stress program will run until we quit with Ctrl+C._
	
Next, view the Metrics page for that VM in Kibana and comparing 2 of web servers to see the differences in CPU usage, confirmed that `metricbeat` is capturing the increase in CPU usage due to our stress command:

![cpu stress test results](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/cpu%20stress%20test%20results.png)


Another view of the CPU usage metrics Kibana collected:

![cpu stress test results graph](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/cpu%20stress%20test%20results%20graph.png)


#### Generate a high amount of web requests to both web servers and make sure that Kibana is picking them up.

This time we will generate a high amount of web requests directed to one of my web servers. To do so, I will use `wget` to launch a DoS attack.

1. Log into my Jump Box Provisioner
	
   - ```bash
        ssh sysadmin@<jump-box-provisioner>
     ``` 

2. We need to add a new firewall rule to allow my Jump Box (10.0.0.4) to connect to my web servers over HTTP on port 80. To do so, I add a new Inbound Security Rule to Red-Team Network Security Group:

![jump to http to webservers](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/jump%20to%20http%20to%20webservers.png)


3. Run the following command to download the file `index.html` from Web-1 VM:

   - ```bash
        wget 10.0.0.5
     ```

Output of the command:

![index html download](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/index%20html%20download.png)


4. Confirm that the file has been downloaded with the `ls` command:


   - ```bash
        sysadmin@Jump-Box-Provisioner:~$ ls 
        index.html
     ```

5. Next, run the `wget` command in a loop to generate a very high number of web requests, I will use the `while` loop:

   - ```bash
        while true; do wget 10.0.0.5; done
     ```

The result is that the `Load`, `Memory Usage` and `Network Traffic` were hit as seen below:

![load increase DoS](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/load%20increase%20DoS.png)

After stopping the `wget` command, I can see that thousands of index.html files were created (as seen below).


![index html files](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/index%20html%20files.png)


I can use the following command to clean that up:

```bash
rm *
```

Now if we use `ls` again, the directory is a lot cleaner:


![directory cleanup](https://github.com/Diablo5G/ELK-Stack-Project/blob/main/Resources/Images/directory%20cleanup.png)


I can also avoid the creation of the `index.html` file by adding the flag `-O` to my command so that I can specify a destination file where all the `index.html` files will be concatenated and written to.

Since I don't want to save the `index.html` files, I will not write them to any output file but instead send them directly to a directory that doesn't save anything, i.e., `/dev/null`. 

I use the following command to do that:


```bash
while true; do wget 10.0.0.5 -O /dev/null; done
```

Now, if I want to perform the `wget` DoS request on all my web servers, I can use the previous command I used to generate failed SSH login attempts on all my web servers, but this time I will tweak the command to send `wget` requests to all webservers:

```bash
while true; do for i in {5..6}; do wget -O /dev/null 10.0.0.$i; done
```

Note that we need to press CTRL + C to stop the `wget` requests since I am using the `while` loop.


My Elastic Stack server is now functioning and correctly monitoring my load-balanced exposed DVWA web application.

</details>

---


### Citations and References:

#### General Resources:

- [`elk-docker` Container Documentation](https://elk-docker.readthedocs.io/)
- [Elastic.co: The Elastic Stack](https://www.elastic.co/elastic-stack)
- [Ansible Documentation](https://docs.ansible.com/ansible/latest/index.html)
- [`elk-docker` Image Documentation](https://elk-docker.readthedocs.io/#elasticsearch-logstash-kibana-elk-docker-image-documentation)
- [Virtual Memory Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/5.0/vm-max-map-count.html#vm-max-map-count)
- [Docker Commands Cheatsheet](https://phoenixnap.com/kb/list-of-docker-commands-cheat-sheet)

#### Azure Documentation:

- Azure's page on peer networks: [Network-Peering](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- Peer networking in Azure How-To: [Global vNet Peering](https://azure.microsoft.com/en-ca/blog/global-vnet-peering-now-generally-available/)
- Microsoft Support: [How to open a support ticket](https://docs.microsoft.com/en-us/azure/azure-portal/supportability/how-to-create-azure-support-request)

---

#### Special thanks:

© Trilogy Education Services, a 2U, Inc., Instructor Jerry Arnold and TAs; Matt McNew, Jansen Russell, Micheal Stephenson.

© The University of Texas at Austin Boot Camp, The Cybersecurity program.

---

