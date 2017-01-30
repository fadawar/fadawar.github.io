---
layout: post
title: Structure of Ansible project
---

In my projects (including [Spine Hero](https://spinehero.com)) I like to use Ansible. 
It's a great tool for deploying your code and setup whole enviroment on server.
With one command you can transform a plain linux server into server 
that has installed newest nginx, gunicorn, postgresql etc. and your application
is running on it.

When you want all of that, you have to write playbooks. Ansible's playbooks are YAML files
which contain tasks. These tasks will setup your server. But when you have tens of tasks
you need to put them into seperates files to maintain readability and modularity.
Because sometimes you want just to deploy your code not to setup whole server, right?
How to do that?

## Structure of project

This is how I organize files inside the root of my project that use Ansible:

```
.
├── ansible/
├── src/
├── .ansible-vault-password.txt
├── ansible.cfg
└── README.md
```

Source code is in `src/` directory and all Ansible related files are placed inside `ansible/`.
There are also two Ansible files in the root. First, `.ansible-vault-password.txt` store password
for Vault files. Vault enables Ansible to use variables from encrypted files. This is great
because you can safely push your encrypted files to git repository
(this also means you can't push .ansible-vault-password.txt to git). Second file, `ansible.cfg`,
is config file for Ansible. In my case it mostly says to Ansible to use `.ansible-vault-password.txt`
for decrypting files.

## Structure of Ansible files

Inside of `ansible/` directory looks like this:

```
ansible/
├── group_vars
│   ├── all.yml
│   ├── production
│   │   ├── vars.yml
│   │   └── vault.yml
│   └── staging
│       ├── vars.yml
│       └── vault.yml
├── roles
│   ├── base
│   │   └── tasks
│   ├── server
│   │   ├── handlers
│   │   ├── tasks
│   │   │   ├── main.yml
│   │   │   ├── setup_db.yml
│   │   │   ├── setup_gunicorn.yml
│   │   │   └── setup_nginx.yml
│   │   └── templates
│   │       ├── gunicorn_start.j2
│   │       └── nginx_site_config.j2
│   └── web
│       ├── tasks
│       └── templates
├── hosts_production
├── hosts_vagrant
├── production.yml
└── vagrant.yml
```

### host files
`hosts_production, hosts_vagrant` - addresses of servers (can be IP)

`production.yml, vagrant.yml` - desc


### group_vars

### roles
