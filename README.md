# Ansible Role Dot Files
An ansible role used to install a dot files repository

## Requirements

This role requires ```git``` on the client to clone the repositories.
If the repository is private, the Python ```pexpect``` module is also required.
See the example playbook for suggestions on intalling the requirements.

## Role Variables

These are required variable for public/private repositories
```yaml
dotfiles_repo: 'https://github.com/MrXcitement/dot-git.git'
dotfiles_dest: '~/github/mrxcitement/dot-git'
dotfiles_version: main
```

These are optional variables for private repositories.
```yaml
dotfiles_username: username
dotfiles_password: secret
```

## Dependencies

- geerlingguy.git
  [Jeff Geerling's git role](https://galaxy.ansible.com/geerlingguy/git)

## Example Playbook

Here is a playbook tnd related files that will use this role to install a
public and private dot-file repo.  The ```secret.yml``` file should be
encrypted using ansible-vault and the password stored in a secure file that is
not in version control.

### secret.yml
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
    dest: 'github/mrxcitement/dot-public'
    version: main
  - repo: 'https://gitlab.thebarkers.com/mike/dot-private'
    dest: 'gitlab/mike/dot-private'
    version: master
    username: the_user
    password: '{{ gitlab_token }}'
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
      become: true
      apt: update_cache=yes cache_valid_time=600
      when: ansible_os_family == 'Debian'

    - name: "Install git"
      become: true
      include_role:
        name: geerlingguy.git

    - name: "Install pexpect"
      become: true
      pip:
        name: pexpect

  tasks:
    - name: "Install dotfiles"
      include_role:
        name: mrxcitement.dotfiles
      vars:
        dotfiles_repo: '{{ item.repo }}'
        dotfiles_dest: '/home/{{ ansible_user }}/{{ item.dest }}'
        dotfiles_version: '{{ item.version }}'
        dotfiles_username: '{{ item.username | default("") }}'
        dotfiles_password: '{{ item.password | default("") }}'
      loop: '{{ dotfiles }}'
``` 

### requirements.yml
```yaml
- src: geerlingguy.git
- src: mrxcitement.dotfiles

## License

MIT

## Author Information

Mike Barker
https://galaxy.ansible.com/mrxcitement
