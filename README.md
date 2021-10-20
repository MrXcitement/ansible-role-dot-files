# Ansible Role Dot Files
An ansible role used to install a dotfiles repository.

The dotfiles repository will be cloned to the dotfiles destination directory.
The dotfiles destination directory will be created in the current users home directory.
The ```install``` script in the dotfiles destination directory will be run via the shell module.

A dotfiles repository has the following convention:
- Contains a ```home``` directory at the root of the repository.
  - This directory holds files and/or directories to be *installed* in the current users home directory.
- Contains an ```install``` shell script at the root of the repository.
  - The ```install``` script will *install* files in the home folder to the current users home folder.
  - The ```install``` script will not display any output if it does not *install* any files.

## Requirements

This role requires ```git``` on the client to clone the repositories.
For private repositories, the Python ```pexpect``` module is also required.
See the example playbook for suggestions on intalling the requirements.

## Role Variables

These are required variables for public/private repositories
```yaml
dotfiles_repo: 'https://github.com/MrXcitement/dot-git.git'
dotfiles_dest: '~/github/mrxcitement/dot-git'
dotfiles_version: main
```

These are optional variables, but required for private repositories.
```yaml
dotfiles_username: the_user
dotfiles_password: the_password
```

## Dependencies

None

## Example Playbook

Here is a playbook and related files that will use this role to install a
public and private dot-file repo. 

The ```secret.yml``` file should be encrypted using ansible-vault and the
password stored in a secure file that is not in version control.

Note: dotfiles will be installed as the current user, so if you ```become:
yes``` before running this role, they will be installed in a '/home/root'
directory.

### secret.yml
*only needed if your accessing a private git repo*
```yaml
---
gitlab_username: 'the_user'
gitlab_password: 'the_password'
```

### vars.yml
```yaml
---
dotfiles:
  - repo: 'https://github.com/the_user/dot-public.git'
    dest: 'github/the_user/dot-public'
    version: main
  - repo: 'https://gitlab.com/the_user/dot-private'
    dest: 'gitlab/the_user/dot-private'
    version: master
    username: '{{ gitlab_username }}'
    password: '{{ gitlab_password }}'
```

### playbook.yml
```yaml
---
- name: Example
  hosts: all

  vars_files:
    - secrets.yml
    - vars.yml

  pre_tasks:
    - name: Update apt cache.
      apt: update_cache=yes cache_valid_time=600
      become: yes
      when: ansible_os_family == 'Debian'

  tasks:
    - name: Install prerequisites
      block:
      - name: Install pexpect
        pip:
          name: pexpect
          
      - name: Install git
        import_role:
          name: geerlingguy.git
      become: yes

    - name: Install dotfiles
      include_role:
        name: mrxcitement.dotfiles
      vars:
        dotfiles_repo: '{{ item.repo }}'
        dotfiles_dest: "{{ item.dest }}"
        dotfiles_version: '{{ item.version }}'
        dotfiles_username: '{{ item.username | default("") }}'
        dotfiles_password: '{{ item.password | default("") }}'
      loop: '{{ dotfiles }}'
``` 

### requirements.yml
```yaml
- src: geerlingguy.git
- src: mrxcitement.dotfiles
```

## License

MIT

## Author Information

Mike Barker
https://galaxy.ansible.com/mrxcitement
