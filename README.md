# Ansible Guide for AWS EC2 Instance Provisioning

## Introduction

This guide provides a step-by-step approach to provisioning an AWS EC2 instance using Ansible. It covers installing necessary dependencies, setting up Ansible Vault for secure credential management, and making the playbook reusable with variables.

This guide includes:
✅ Installing dependencies (Boto3 and AWS Collection)  
✅ Writing an Ansible playbook for EC2 provisioning  
✅ Using Ansible Vault for credential security  
✅ Making the playbook reusable with variables  
✅ Running the playbook to create an EC2 instance  

---

## Installing Dependencies

Before provisioning AWS resources, install the required dependencies:

```sh
pip install boto3 botocore
ansible-galaxy collection install amazon.aws
```

- **Boto3**: Python SDK for AWS, required for Ansible’s AWS modules.
- **Amazon AWS Collection**: Collection of Ansible modules for AWS provisioning.

Verify installation:

```sh
ansible-galaxy collection list | grep amazon.aws
```

---

## Writing an Ansible Playbook to Create an EC2 Instance

### 1️⃣ Creating the Role for EC2 Instance

First, initialize an Ansible role for EC2 provisioning:

```sh
ansible-galaxy role init ec2
```

Inside `ec2/tasks/main.yaml`, add the following task to create an EC2 instance:

```yaml
#SPDX-License-Identifier: MIT-0
---
  - name: start an instance with a public IP address
    amazon.aws.ec2_instance:
      name: "ansible-instance"
      instance_type: t3.micro
      security_group: default
      region: eu-north-1
      aws_access_key: "{{ ec2_access_key }}"  # From vault as defined
      aws_secret_key: "{{ ec2_secret_key }}"  # From vault as defined
      network:
        assign_public_ip: true
      image_id: ami-016038ae9cc8d9f51
      tags:
        Environment: Testing
```

### 2️⃣ Writing the Playbook to Use the Role

Create a playbook `ec2-creation-playbook.yaml` to apply the role:

```yaml
---
- hosts: localhost
  connection: local
  roles:
    - ec2
```

---

## Securing AWS Credentials with Ansible Vault

To store sensitive AWS credentials securely, use Ansible Vault.

### 1️⃣ Creating a Vault Password File

Generate a strong password and store it:

```sh
openssl rand -base64 2048 > vault.pass
```

### 2️⃣ Storing AWS Credentials Securely

Run the following command to create an encrypted variable file:

```sh
ansible-vault create group_vars/all/pass.yml --vault-password-file vault.pass
```

Inside `pass.yml`, define the credentials:

```yaml
---
ec2_access_key: "YOUR_AWS_ACCESS_KEY"
ec2_secret_key: "YOUR_AWS_SECRET_KEY"
```

### 3️⃣ Running the Playbook with Vault

Execute the playbook while providing the vault password:

```sh
ansible-playbook ec2-creation-playbook.yaml --vault-password-file vault.pass
```

---

## Making the Playbook Reusable with Variables

Instead of hardcoding values, use variables.

### 1️⃣ Modifying the Task to Use Variables

Modify `ec2/tasks/main.yaml` to reference variables:

```yaml
#SPDX-License-Identifier: MIT-0
---
  - name: start an instance with a public IP address
    amazon.aws.ec2_instance:
      name: "ansible-instance"
      instance_type: "{{ type }}"
      security_group: default
      region: eu-north-1
      aws_access_key: "{{ ec2_access_key }}"  # From vault as defined
      aws_secret_key: "{{ ec2_secret_key }}"  # From vault as defined
      network:
        assign_public_ip: true
      image_id: ami-016038ae9cc8d9f51
      tags:
        Environment: Testing
```

### 2️⃣ Creating a Separate Role for Variables

To store variable defaults, create another role:

```sh
ansible-galaxy role init ec2-variables
```

Modify `ec2-variables/defaults/main.yaml` to define default values:

```yaml
---
type: t3.micro
```

### 3️⃣ Writing the New Playbook

Create `ec2-creation-variables-playbook.yaml` to use both roles:

```yaml
---
- hosts: localhost
  connection: local
  roles:
    - ec2-variables
```

Run the playbook with:

```sh
ansible-playbook ec2-creation-variables-playbook.yaml --vault-password-file vault.pass
```

---

## Conclusion

By following this guide, you can:
✅ Install dependencies and set up Ansible for AWS  
✅ Write an Ansible playbook to create an EC2 instance  
✅ Secure AWS credentials with Ansible Vault  
✅ Use variables to make the playbook reusable  

This approach makes your AWS provisioning scalable, modular, and secure. 🚀