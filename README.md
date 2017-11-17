# Ansible - AWS EC2 Instances

Demo Ansible Playbook for creating, stopping, terminating the EC2 instance

  - Create a custom Security Group in AWS to allow ports 22 (SSH), 80 (HTTP) and ICMP.
  - Create/Stop/Terminate the EC2 instance - Using Ansible EC2 module
  - Wait for the SSH is up on Instance.

## Installation

Firstly, you need to install and configure AWS CLI (if you don't already have it). Depending on your operating system and environment, there are many ways to install AWS CLI. You can read this article to setup AWS CLI [How to install and configure AWS CLI](http://devopsmates.com/install-configure-aws-cli-amazon-web-services-command-line-interface/).

#### 1. Create key_pair with name "AWS-Ansible"

```sh
ansible@ansible-node:~/ec2-instance$ aws ec2 create-key-pair --key-name "AWS-Ansible"
```

#### 2. Create Security Group in AWS to allow ports 22/SSH, 80/HTTP and ICMP.



```
---
- hosts: server
  tasks:
    - name: Setting up the Security Group for new instance
      ec2_group:
          name: Ansible_Security_Group_AWS
          description: Allowing Traffic on port 22 and 80
          region: ap-southeast-1
          rules:
           - proto: tcp
             from_port: 80
             to_port: 80
             cidr_ip: 0.0.0.0/0

           - proto: tcp
             from_port: 22
             to_port: 22
             cidr_ip: 0.0.0.0/0

           - proto: icmp
             from_port: -1
             to_port: -1
             cidr_ip: 0.0.0.0/0
          rules_egress:
           - proto: all
             cidr_ip: 0.0.0.0/0
          vpc_id: vpc-d8826ebd

    - name: Provision EC2 instance
      ec2:
         key_name: AWS-Ansible
         region: ap-southeast-1
         instance_type: t2.micro
         image: ami-67a6e604
         wait: yes
         wait_timeout: 500
         count: 1
         instance_tags:
            Name: AWS-Ansible
         volumes:
            - device_name: /dev/xvda
              volume_type: gp2
              volume_size: 8
         monitoring: yes
         vpc_subnet_id: subnet-18e7fa93
         assign_public_ip: yes
         group: Ansible_Security_Group_AWS
      register: ec2

    - name: Wait for SSH to come up
      wait_for:
          host: "{{ item.public_dns_name }}"
          port: 22
          delay: 60
          timeout: 320
          state: started
      with_items: "{{ ec2.instances }}"
```

#### 3. Execution Ansible playbooks to create EC2 instance

```sh
ansible@ansible-node:~/ec2-instance$ ansible-playbook -i hosts main/ec2-creation.yml

PLAY [server] ****************************************************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************************************************************************************************************************************************************************
ok: [127.0.0.1]

TASK [Setting up the Security Group for new instance] ************************************************************************************************************************************************************************************************************************
ok: [127.0.0.1]

TASK [Provision EC2 instance] ************************************************************************************************************************************************************************************************************************************************
changed: [127.0.0.1]

TASK [Wait for SSH to come up] ***********************************************************************************************************************************************************************************************************************************************
ok: [127.0.0.1] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-172-30-0-21.ap-southeast-1.compute.internal', u'public_ip': u'13.228.168.45', u'private_ip': u'172.30.0.21', u'id': u'i-0a2ae87d8b3759b98', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/xvda': {u'status': u'attached', u'delete_on_termination': False, u'volume_id': u'vol-0137dbee68f8e07a2'}, u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-04734a1c59869218d'}}, u'key_name': u'RML-AWS-Ansible', u'image_id': u'ami-67a6e604', u'tenancy': u'default', u'groups': {u'sg-db7531bd': u'Ansible_Security_Group_AWS'}, u'public_dns_name': u'ec2-13-228-168-45.ap-southeast-1.compute.amazonaws.com', u'state_code': 16, u'tags': {u'Environment': u'RML', u'Name': u'RML-AWS-Ansible'}, u'placement': u'ap-southeast-1a', u'ami_launch_index': u'0', u'dns_name': u'ec2-13-228-168-45.ap-southeast-1.compute.amazonaws.com', u'region': u'ap-southeast-1', u'launch_time': u'2017-11-17T03:16:15.000Z', u'instance_type': u't2.micro', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'})

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
127.0.0.1                  : ok=4    changed=1    unreachable=0    failed=0
```

#### 4. Stop the EC2 instance

```
ansible@ansible-node:~/ec2-instance$ ansible-playbook -i hosts main/ec2-stop.yml

PLAY [server] ****************************************************************************************************************************************************************************************************************************************************************

TASK [Stop EC2 instance] *****************************************************************************************************************************************************************************************************************************************************
changed: [127.0.0.1]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
127.0.0.1                  : ok=1    changed=1    unreachable=0    failed=0
```

#### 5. Terminate the EC2 instance

```
ansible@ansible-node:~/ec2-instance$ ansible-playbook -i hosts main/ec2-terminate.yml

PLAY [server] ****************************************************************************************************************************************************************************************************************************************************************

TASK [Terminate EC2 instance] ************************************************************************************************************************************************************************************************************************************************
changed: [127.0.0.1]

PLAY RECAP *******************************************************************************************************************************************************************************************************************************************************************
127.0.0.1                  : ok=1    changed=1    unreachable=0    failed=0
```

