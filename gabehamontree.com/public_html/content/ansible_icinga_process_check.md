Title: Verify Icinga is Running with Ansible!
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example
Slug: ansible_icinga_check
Authors: Gabe Hamontree
Summary: Verifying Icinga is Running with Ansible
Status: published


```
---
- hosts: test
  become: true
  tasks:

    - name: Restart if needed & add enabled
      shell: |
        if ps -axef | egrep 'icinga' | grep -v grep > /dev/null; then
            echo 'Icinga is running'
        else
            echo 'Icinga not running'
            systemctl start icinga
            echo 'Started Icinga'
            systemctl enable icinga
            echo 'Enabled Icinga'
        fi
      ignore_errors: false
      register: icinga_process_check
    - debug:
        var: icinga_process_check.stdout
```