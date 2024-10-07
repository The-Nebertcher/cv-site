Title: Swap NTP Config with Ansible!
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example, ntp, ansible
Slug: ansible_ntp_config
Authors: Gabe Hamontree
Summary: Swap NTP Config with Ansible
Status: published


```
---
- hosts: test
  become: true
  tasks:

    - name: Stop ntp
      shell: | 
        systemctl stop ntpd 

    - name: Backup old config
      shell: | 
        mv /etc/ntp.conf /etc/ntp.conf.{{ ansible_date_time.iso8601 }}

    - name: Add new lines
      shell: | 
        echo "restrict default kod nomodify notrap nopeer noquery" >> /etc/ntp.conf 
        echo "restrict -6 default kod nomodify notrap nopeer noquery" >> /etc/ntp.conf 
        
        echo "restrict 127.0.0.1" >> /etc/ntp.conf 
        echo "restrict -6 ::1" >> /etc/ntp.conf 
        
        echo "server 10.0.123.10 iburst #ntp01.slc01" >> /etc/ntp.conf 
        echo "server 10.0.123.11 iburst #ntp02.slc01" >> /etc/ntp.conf 
        echo "server 10.0.123.12 iburst #ntp03.slc01" >> /etc/ntp.conf 
        echo "server 10.1.123.10 iburst #ntp01.ntp" >> /etc/ntp.conf 
        echo "server 10.1.123.11 iburst #ntp02.ntp" >> /etc/ntp.conf 
        
        echo "server	127.127.1.0	# local clock" >> /etc/ntp.conf 
        echo "driftfile /var/lib/ntp/drift" >> /etc/ntp.conf 
        echo "keys /etc/ntp/keys" >> /etc/ntp.conf 
        echo "disable monitor" >> /etc/ntp.conf 


    - name: Restart ntp & Enable
      shell: | 
        systemctl restart ntpd 
        systemctl enable ntpd
```