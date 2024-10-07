Title: Run Patches With Ansible!
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example, yum, ansible
Slug: ansible_upgrade_package
Authors: Gabe Hamontree
Summary: Run Patches With Ansible
Status: published

```
- hosts: test
  become: yes
  tasks:

    - name: Grab unit files for comparison
      shell: | 
        rm -f /tmp/services
        systemctl list-units >> /tmp/services

    - name: Run Yum update
      yum:
        name: '*'
        state: latest

    - name: Reboot
      reboot:
      #exclude:
      #register: response

    #- debug: msg="{{ response.stdout }}"
```