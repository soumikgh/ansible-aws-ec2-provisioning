AWS ec2 provision
=================

This ansible role creates, terminates, start or stop an instance in ec2. 

Requirements
------------

- Ansible 2.0.1 or higher.
- Tested on Ubuntu 14.04, CentOS 7 and Amazon 7

Role Variables
--------------

| parameter             | required | default | choices | comments |
| --------------------- | -------- | ------- | -------- |-------- |
| ec2_find_ami_name                   |  no     |         || ami name to find. Ex: ubuntu-docker-base-ami-1* |
| ec2_assign_public_ip                   |   no    |    yes     |yes, no|  when provisioning within vpc, assign a public IP address. |
| ec2_count                   |  no     |     1    || number of instances to launch|
| ec2_sg_id                   |   yes    |         || security group id (or list of ids) to use with the instance |
| ec2_instance_type                   |   no    |     t2.micro    || instance type to use for the instance, see http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-types.html|
| ec2_ebs_optimized  | no  |  false ||  whether instance is using optimized EBS volumes, see http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSOptimized.html|
| ec2_instance_profile_name |  no |   ||  Name of the IAM instance profile to use. Recommended to use aws-elasticbeanstalk-ec2-role |
| ec2_base_image  |  yes |   ||  ami ID to use for the instance. Only use when ec2_find_ami_name is blank. |
| ec2_key_name | yes  |   |   |key pair to use on the instance|
| ec2_monitoring  |  no | yes  | yes, no |  enable detailed monitoring (CloudWatch) for instance | 
| ec2_vpc_subnet_id | yes  |   || the subnet ID in which to launch the instance (VPC)  |
| ec2_user_data | no | | | opaque blob of data which is made available to the ec2 instance |
| ec2_volumes    | no | | |a list of hash/dictionaries of volumes to add to the new instance; '[{"key":"value", "key":"value"}]'; keys allowed are - device_name (str; required), delete_on_termination (bool; False), device_type (deprecated), ephemeral (str), encrypted (bool; False), snapshot (str), volume_type (str), iops (int) - device_type is deprecated use volume_type, iops must be set when volume_type='io1', ephemeral and snapshot are mutually exclusive.  |
| aws_resource_tags  | yes  |   | | a hash/dictionary of tags to add to the new instance or for starting/stopping instance by tag; '{"key":"value"}' and '{"VREnv":"PROD","VRProject":"sample","VRTeam":"infra", "Name":"instance_name"}' |
| state |  no |  present |present, absent,running, stopped| create or terminate instances  |
| region |  yes |   || The AWS region to use. Must be specified if ec2_url is not used. If not specified then the value of the EC2_REGION environment variable, if any, is used. See http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region  |
| vivareal_project_build | yes | | | ec2 instance name |
| instance_ids | no | | | ec2 instance id's, required when setting state to absent |


Ansible modules
---------------
[ec2](http://docs.ansible.com/ansible/ec2_module.html)

[ec2_ami_find](http://docs.ansible.com/ansible/ec2_ami_find_module.html)

Output variables
----------------
    ec2_ami_instance_id: launched instance id
    ec2_launched: registered host group

Example Playbook (creation)
---------------------------

   
    - hosts: localhost
      vars:
        aws_resource_tags: {
         'Name': 'instance-name',
         'VREnv': 'PROD',
         'VRProject': 'infra-ansible',
         'VRTeam': 'infra',
         'zabbix-metadata': 'if-ansible-sample' }
        region: us-east-1
        ec2_sg_id: ['sg-32f1634b']
        ec2_vpc_subnet_id: subnet-0959b37f
        ec2_vpc_id: vpc-202e7845
        ec2_key_name: master_virginia
        ec2_assign_public_ip: yes
        ec2_find_ami_name: "ubuntu-docker-base-ami-1*"
        ec2_instance_type: t2.small
        ec2_instance_profile_name: aws-elasticbeanstalk-ec2-role
        ec2_base_image: ami-8fcee4e5
        ec2_user_data:|
            docker run nginx
        vivareal_project_name: my-ami
        vivareal_build_version: 1
        vivareal_project_build: "{{ vivareal_project_name }}-{{ vivareal_build_version}}"
        ec2_volumes:
          - device_name: /dev/sda1
            volume_type: gp2
            volume_size: 200
            delete_on_termination: true
      roles:
        - { role: aws-ec2-provisioning }


Example Playbook (termination)
------------------------------


    - hosts: localhost
      vars:
        state: absent
        region: eu-west-1
        instance_ids: ['i-0eef911d60fe285cf' ]
      roles:
        - { role: aws-ec2-provisioning } 



License
-------

BSD

Author:
------------------

Giancarlo Rubio (<gianrubio@gmail.com>)
