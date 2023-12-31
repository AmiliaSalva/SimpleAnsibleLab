# Simple Ansible Automation Lab with Linode

![Dark Purple Technical Roadmap Brainstorm](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/42a5394e-409c-4b64-8b8f-77c8c5fea5f1)


## Introduction

To preface this, I wanted to mention my journey into IT has always been driven by an insatiable curiosity and a hunger to understand the tools and practices that shape our digital world. This lab, in particular, stems from my profound desire to constantly learn new things and technologies. 

Not just to know how it functions but to understand its core principles and nuances. While there are numerous tutorials and courses, there's no better way to learn than by doing. By setting up this lab, I aim to practically apply Ansible concepts, experiment, make mistakes, learn, and gain a robust understanding of this remarkable automation tool. 

Leveraging Linode as our cloud platform, we'll use Ansible to install and manage software across multiple virtual machines, taking you through each step of my exploration journey.

. This lab is meant as a simple introduction, aiming to showcase just a glimpse of what you can achieve with Ansible. 



## Objectives of the Lab:
  - **Introduction to Automation:** Demonstrate the importance and benefits of automation in the modern IT world. Showcase how tasks that traditionally took hours can be accomplished in minutes with a few scripts.
  - **Hands-on with Ansible:** Understand how to set up a master node, write playbooks, and dispatch commands to worker nodes.
  - **Infrastructure as Code with Linode:** Highlight the agility and flexibility of cloud hosting with Linode. Learn to spin up, manage, and tear down virtual servers programmatically.
  - **SSH and Remote Management with Solar PuTTY:** Understand the basics of secure shell (SSH) connections and the convenience brought by tools like Solar PuTTY.
  - **Package Management with YUM:** Get acquainted with software package management on CentOS using YUM. Understand how software can be installed, updated, and removed seamlessly.
  - **End-to-End Workflow:** By the end of the lab, participants should be able to set up a new virtual server environment, install and configure Ansible, connect to worker nodes, and run automation playbooks—all orchestrated through code.
    
## Technologies Used

  - **Ansible:**
        A leading open-source automation tool, Ansible is used to automate tasks such as software installations, configuration management, and even complex multi-tier application deployments.

  - **Linode:**
        A versatile cloud hosting provider. For our lab, Linode provided the infrastructure, allowing us to spin up and manage virtual servers on-demand. For cost efficiency and given the nature of the lab, we deployed servers with Linode's most affordable plan, as our activities didn't demand extensive resources.

  - **CentOS:**
        A popular Linux distribution. Our virtual servers (both master and worker nodes) ran CentOS, providing a consistent environment for our automation tasks.

  - **Solar PuTTY:**
        An evolution of the traditional PuTTY terminal. Solar PuTTY provides a more user-friendly interface, allowing us to SSH into our virtual servers seamlessly.

  - **YUM:**
        The package manager for our CentOS servers. Through YUM, we managed software packages, ensuring the latest versions were installed and dependencies were met.

 <br />
 <br />

## Lab Setup:

For this lab, we've leveraged Linode to set up 3 cloud servers. One of these servers will act as the control station or "master", from which Ansible commands will be dispatched. The other two servers will be our "worker" nodes, which will receive commands and configurations from the master.

### The results should look like this:

![overall Lab](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/b9546bdb-6a2b-4fe5-b938-da0f285f5d6b)


 <br />


## Control Station Setup:

After deploying our control station on Linode, we initiate it with the following bash script:



    #!/bin/bash

    yum update -y
    yum install epel-release -y
    yum install ansible -y

## What does this script do?

- `yum update -y`: Updates all packages on the system to the latest version.
- `yum install epel-release -y`: Installs the EPEL (Extra Packages for Enterprise Linux) repository, providing extra packages for CentOS.
- `yum install ansible -y`: Installs Ansible, our automation tool.



### After this, two additional CentOS servers will be deployed, acting as our worker nodes:
 

 
  <br />

- **Creating and Deploying Servers in Linode**
  
![Server Creation](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/b95f3c9e-3aa2-4fe4-a800-3538ec4e5ab7)
  
  <br />

- **Configuring the server to the specifications needed for this lab.**

   <br />
   
![Linode Server Selection](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/848427c9-a4b0-4f2d-94cb-943cc5ac0599)




## Configuration:

With the servers up and running, we'll use Solar PuTTY to SSH into the control station. Once logged in, navigate to the Ansible configuration directory:



``cd /etc/ansible``

Inside this directory, you'll find key configuration files: ``ansible.cfg`` and hosts.
Setting Up the Hosts:

Edit the hosts file with the following command:
**(Note: If you're not logged in as root, prefix the command with ``sudo``.)**


    vi hosts



Within the ``hosts`` file:

- Define a group for our worker nodes:

 

      [linux]
      IP_ADDRESS_1
      IP_ADDRESS_2

Beneath this, paste the IP addresses of the two worker nodes.

Configure group variables:

    [linux:vars]
    ansible_user=root
    ansible_password=YOUR_DEFINED_PASSWORD

<br />
 
### It should look something like this in the SSH client:



![sshclientexample](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/3f93b678-0468-4cb7-a746-d3184a913256)




 <br />
 <br />

## Why do this?

  - Grouping allows you to target specific sets of machines. Here, ``[linux]`` is our group of worker nodes.
  - ``ansible_user`` and ``ansible_password`` define the login credentials Ansible will use when connecting. For the purpose of this lab, we're using ``root`` (though this isn't recommended for production setups).

Remember to save and quit ``vi:`` Press ``Esc``, then type ``:wq``.

## Configuring Ansible:

Edit ``ansible.cfg`` by using the following command:

    vi ansible.cfg

Find the line ``#hostkey_checking=false`` and uncomment it. This will disable SSH host key checking for the lab (this is not recommended for production environments, but for the purpose of this lab, it is fine.). Save and exit.

## Testing Connectivity:

### Test the configuration with the following command:


    ansible linux -m ping

This pings the worker nodes to check if they're reachable.

### The results should look like this:

![ansible ping](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/1adb5865-229c-4628-b957-087f896f23cb)


## Verify OS details of worker nodes:


    ansible linux -a "cat /etc/os-release"

This retrieves and displays the OS details of the worker nodes.

### The results should look like this:

![more ansible commands](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/36e4e458-92f6-4101-9441-ba28aa761276)


## Utilizing an Ansible Playbook:

Playbooks are Ansible’s configuration, deployment, and orchestration language. For this lab, we'll set up a simple playbook to ensure ``nano`` is installed.

### Create a new playbook using the following command:


    vi iluvnano.yml

### Once created, insert the following YAML script:


    ---
    - name: iluvnano
      hosts: linux
      tasks:
        - name: ensure nano is there
          yum:
            name: nano
            state: latest

## To run the playbook:


    ansible-playbook iluvenano.yml

This will install or ensure the latest version of nano is present on the worker nodes.

## The results should look similar to this:

![Screenshot 2023-08-13 at 09-40-56 ChatGPT](https://github.com/AmiliaSalva/SimpleAnsibleLab/assets/132176058/7d86cf5c-1b33-407d-92d8-258a0be00db4)

- **Gathering Facts:** Ansible, by default, gathers facts about the remote hosts before executing any tasks. This phase collects information about the remote system, like its OS, IP addresses, memory stats, etc. Both servers (``linux_server_1`` and ``linux_server_2``) successfully responded.
- **ensure nano is there:** For ``linux_server_1``, the changed status indicates that ``nano`` was either not present or not at the latest version, and Ansible took corrective action to ensure it is the latest.
    For ``linux_server_2``, the ok status indicates that nano was already at the latest version, so no changes were made.
- **PLAY RECAP:** This is a summary of what happened. The ``ok``, ``changed``, and other counters reflect the number of tasks that were executed, how many resulted in changes, if any were unreachable, etc.



## Conclusion:

By integrating these technologies and outlining clear objectives, I aim to present a simple yet insightful peek into the realm of modern IT automation. With an emphasis on cost-effective learning, I took advantage of Linode's budget-friendly plans, ensuring that quality wasn't compromised. My aspiration is for you to walk away with a heightened curiosity about automation and the powerful tools behind it, especially Ansible.
