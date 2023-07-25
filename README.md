# Ansible playbook for my NixOS cluster
### This will download the current configuration.nix files from each node as a backup, then upload a common configuration file and test it, and switching to it and rebooting if it passes.
---
This is the host names I use in Docker swarm:
- monroe_nodes == Manager nodes
- monet_nodes == Worker nodes
```yaml
---
- name: Backup existing configs - Upload and test new configs and rebuild switch to them, with a reboot at the end
  hosts: monroe_nodes monet_nodes
  become: yes

  tasks:
    - name: Download backups of existing configuration.nix files
      ansible.builtin.fetch:
        src: /etc/nixos/configuration.nix

        # Final format: ./backup_configs/hostname-25-07-2023 kl 15:19
        dest: ./backup_configs/{{ansible_hostname}}-{{ '%d-%m-%Y kl %H:%M' | strftime }}/configuration.nix
        flat: yes

    - name: Upload local configuration.nix to remotes
      ansible.builtin.copy:
        src: ./configuration.nix
        dest: /etc/nixos/configuration.nix
        # A backup is made on each remote as well. Might remove this later.
        backup: yes

    - name: Test config
      ansible.builtin.shell:
        cmd: nixos-rebuild test

    - name: rebuild switch, if test succesful
      ansible.builtin.shell:
        cmd: nixos-rebuild switch

# To make the Ansible reboot module work for NixOS
    - name: Reboot remotes for full config initialization
      ansible.builtin.reboot:
        search_paths:
          - '/run/current-system/sw/bin'
```
