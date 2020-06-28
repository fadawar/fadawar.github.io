.. title: Structure of Ansible project
.. slug: structure-of-ansible-project
.. date: 2017-01-31 18:38:03 UTC+02:00
.. tags: ansible, deployment
.. category: 
.. link: 
.. description: 
.. type: text

In my projects (including [Spine Hero](https://spinehero.com)) I like to use Ansible. 
It's a great tool for deploying your code and for setup of whole enviroment on server.

<!-- TEASER_END -->

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

```html
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

```html
ansible/
├── group_vars
│   ├── all.yml
│   ├── production
│   │   ├── vars.yml
│   │   └── vault.yml
│   └── staging
│       ├── vars.yml
│       └── vault.yml
├── roles
│   ├── base
│   │   └── tasks
│   ├── server
│   │   ├── handlers
│   │   ├── tasks
│   │   │   ├── main.yml
│   │   │   ├── setup_db.yml
│   │   │   ├── setup_gunicorn.yml
│   │   │   └── setup_nginx.yml
│   │   └── templates
│   │       ├── gunicorn_start.j2
│   │       └── nginx_site_config.j2
│   └── web
│       ├── tasks
│       └── templates
├── hosts_production
├── hosts_vagrant
├── production.yml
└── vagrant.yml
```

### host files
`hosts_production, hosts_vagrant` - addresses of servers (can be IP) with login information

`production.yml, vagrant.yml` - contain main task and variables


### group_vars
This directory contains variables that are used in tasks. 

`all.yml` - General variables that are same on production and also staging - like application name, server name

`production/vars.yml` - Variables that are specific for the enviroment - different paths, enable/disable debug mode, ssl

`production/vault.yml` - encrypted file with variables like passwords to database and etc

### roles
Inside of `roles/` directory are placed playbooks with tasks. They are grouped together into directories 
by purpose of task they contain.

`base/` - Installation of general packages that are need to monitor and work on the server.

`server/` - Every task you need to setup a web server. Installation of nginx, postgresql, memcached, etc.
Configurations for this tools are in `templates/` directory. Templates use variables from `group_vars` directory.

`web` - Tasks focused on deploying a new version of your app, running migrations and things you need 
to achieve zero downtime.

## Provisioning with one command

Now, when you have all of this you can deploy your app to a new server with one command:

```sh
ansible-playbook -i ansible/hosts_production ansible/production.yml
```

_Do you have any questions? Write them in comments belove._

