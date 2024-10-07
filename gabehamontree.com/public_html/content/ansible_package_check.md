Title: Get Info About a Package With Ansible!
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example, yum, ansible
Slug: ansible_check_package
Authors: Gabe Hamontree
Summary: Get Info About a Package With Ansible
Status: published

```
---
- hosts: localhost
  gather_facts: False
  vars_prompt:
  - name: package
    prompt: "What is the name of the package you want to know the version of?"
    private: no

  tasks:
    - name: Check specific package version
      shell: "yum info {{ package }} >> /tmp/output && cat /tmp/output | grep 'Version     : ' && rm -f /tmp/output"
      ignore_errors: yes
      register: result
    - debug:
        msg: "{{ package }} is on {{ result.stdout }}"
```