Title: Get CRON Output and Check for Services With Ansible
Date: 2023-12-28 10:20
Category: Ansible
Tags: blog, example
Slug: ansible_cron_check
Authors: Gabe Hamontree
Summary: Verifying CRON, RSYNC, SSH Keys with Ansible
Status: published


```
    ---
    - hosts: test
      become: true
      tasks:
    
        - name: Grab cron output
          shell: | 
            for i in /var/spool/cron/*  ; do export f="$(basename $i)" ; grep -vEi "^#|^mailto|^$" "$i" | awk -F '\n' '{print ENVIRON["f"] "," $1}' ; done >>/tmp/{{ ansible_fqdn }}_{{ inventory_hostname }}
          register: cron_result
    
        - set_fact:
            cron_result={{ cron_result.stdout }}
    
        - debug: var=cron_result
          run_once: true
    
        - name: Create remote dir on the ansible host box
          ignore_errors: yes
          shell: |
            mkdir /var/www/html/ansible/cron_output/{{ansible_fqdn}}
          delegate_to: 127.0.0.1
    
        - name: Make sure rsync is installed
          become: true
          become_user: root
          action: >
            {{ ansible_pkg_mgr }} name=rsync state=present update_cache=yes
    
        - name: ensure ssh-key is present
          ansible.posix.authorized_key:
            user: "{{ ansible_user_id }}"
            state: present
            key: "{{ lookup('file', item) }}"
          with_fileglob:
          - /{{ ansible_user_id }}/.ssh/*.pub
          delegate_to: 127.0.0.1
    
        - name: rsync the output file to the ansible box
          ansible.posix.synchronize: src={{inventory_hostname}}/tmp/{{ ansible_fqdn }}_{{ inventory_hostname }} dest=/var/www/html/ansible/cron_output/{{ansible_fqdn}} dirs=yes use_ssh_args=yes
            #mode: pull
            #src:  rsync://{{inventory_hostname}}/tmp/{{ ansible_fqdn }}_{{ inventory_hostname }}
            #dest: /var/www/html/ansible/cron_output/{{ansible_fqdn}}
          delegate_to: 127.0.0.1
    
    
          #shell: | 
          #  rsync -avzhe ssh /tmp/{{ ansible_fqdn }}_{{ inventory_hostname }} root@10.0.0.163:/var/www/html/ansible/cron_output/{{ansible_fqdn}}
```