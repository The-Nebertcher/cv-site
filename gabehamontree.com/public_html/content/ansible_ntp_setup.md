Title: Setup NTP with Ansible!
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example, ntp, ansible
Slug: ansible_ntp_setup
Authors: Gabe Hamontree
Summary: Setup NTP with Ansible
Status: published

```
---
- hosts: test 
  tasks:
  - name: set timezone
    shell: timedatectl set-timezone America/Denver

  - name: Copy over the NTP configuration
    template: src=./templates/ntp.conf dest=/etc/ntp.conf
    notify:
    - restart ntpd
    tags: ntp
 
  - name: Make sure NTP is stopped
    service: name=ntpd state=stopped enabled=yes
    tags: ntp
 
  - name: Sync time initialy
    shell: ntpdate 10.1.0.67
    tags: ntp

  - name: Sync hwclock
    shell: hwclock -w
    tags: ntp

  handlers:
  - name: restart ntpd
    service: name=ntpd state=restarted
```