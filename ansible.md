# Ansible

Ansible is an IT automation tool that can be used to configure systems, deploy software & orchestrate advanced tasks such as CI/CD & also supports zero-downtime rolling updates.

Ansible is __agentless__, meaning that to establish connectivity, it requires basic SSH for Linux systems & Powershell for Windows (no additional software or _agent_ needs to be installed & configured)

Ansible works via _push_ mechanism wherein the controller node pushes updates & configurations to the target servers.

To execute actions on remote systems using Ansible, we need few things:
- __Inventory:__ Information about target systems (Default: `/etc/ansible/hosts)
- __Module:__ A program (usually python) that executes, does some work and returns proper JSON output
- __Task:__ Execution of a single Ansible module
- __Playbook:__ Contains a set of instructions to execute a list of tasks on a group of target hosts

Ansible also supports __idempotency__ (an operation is idempotent if the result of performing it once is exactly the same as the result of performing it repeatedly without any intervening actions)

## Prerequisites:

- A _Controller_ (VM, VPS, etc)
- At least 1 _Target_ (VM, VPS, etc)


## Setup:

On the system chosen to be the controller, do the following:

```bash
$ sudo apt update
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```

To check if Ansible has been installed correctly:
```bash
$ ansible localhost -m ping

# Should return the following:

# localhost | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

To test if controller can connect to target(s):

_Note: If the target requires ssh via public key, ssh into the target from the controller once to add it to the list of known hosts_

```bash
$ mkdir testdir
$ cd testdir

# Create an inventory
$ cat > inventory.txt
<target-name> ansible_host=<target-ip> ansible_ssh_pass=<target-ssh-password>
# You may add more targets

# Call the `ping` module using the inventory above
$ ansible <target-name> -m ping -i inventory.txt

# Should return the following:

# <target-name> | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }

# To ping all the targets, use:
$ ansible all -m ping inventory.txt
```

If you receive the error regarding host key checking, either try first doing manual ssh from controller to target (recommended) or do the following (not recommended for production):
```bash
$ nano /etc/ansible/ansible.cfg
```
 Now, uncomment the line:
 ```bash
# host_key_checking = False
 ```

 For production, one can use:
 ```bash
$ ansible target1 -m ping --private-key=<path-to-ssh-pvt-key> -i inventory.txt
 ```

 Now one can try to control the target servers to get configured. A naive way to do so is shown (such commands are known as __Ad hoc Commands__):

 ```bash
# Install nginx on all target servers (Use the `apt` module [-m] with relevant arguments [-a])
$ ansible all -m apt -a "pkg=nginx state=latest update_cache=true"
```
>Here, `state` can take values: 
> - __absent:__ check the package is not installed
> - __build-dep:__ ensures the package build dependencies are installed
> - __installed:__  ensure that a desired package is installed
> - __latest:__ checks the latest version of package is installed
> - __present:__ checks if a package is installed or not, but does not update it
> - __removed:__ remove the specified package
> - __present:__ ensure that a desired package is installed

For more than 5 servers, one can use `-f <number-of-forks>` argument to set the number of parallel forks (default = 5)

To get a list of all modules available:
```bash
$ ansible-doc -l
```

## Inventories:

An inventory is a set of objects or hosts, against which we are executing our playbooks or single tasks via shell commands.

Syntax of an inventory:
```bash
<alias1> ansible_host=<server-1-ip> ansible_connection=<ssh/winrm/localhost>
<alias2> ansible_host=<server-2-ip> ansible_ssh_pass=<target-ssh-password>

[<server-group-1>]
<alias1>
<alias2>

 # and so on...
```

Sample inventory file:
```bash
# Sample Inventory File

# Web Servers
web1 ansible_host=server1.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web2 ansible_host=server2.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!
web3 ansible_host=server3.company.com ansible_connection=ssh ansible_user=root ansible_ssh_pass=Password123!

# Database Servers
db1 ansible_host=server4.company.com ansible_connection=winrm ansible_user=administrator ansible_password=Password123!

[web_servers]
web1
web2
web3

[db_servers]
db1

[servers:children]
web_servers
db_servers
```

To create a new inventory to be used as default:
```bash
$ cd /etc/ansible/
$ mv hosts hosts.orig
$ nano hosts
# Now fill in the details bout targets in this file & save it
```

## Playbooks

They are [YAML](https://learnxinyminutes.com/docs/yaml/) files where Ansible code is written to tell Ansible what to execute. They form the building blocks of all use-cases of Ansible.

A playbook contains one or more __plays__ which define a set of hosts to configure and a list of tasks to be performed.

A sample playbook:
```yaml
- hosts: all

  tasks:
    - name: "ping all"
      ping:

    - name: "execute a shell command"
      shell: "date; whoami; df -h;"
```
The above inventory would execute on `all` hosts and perform the following tasks sequentially:
- ping the target servers
- execute the 3 shell commands & return output to controller

A more complex playbook to install apache & configure log level _[Log levels are means of categorizing the entries in your log file]_:
```yaml
---
- hosts: all

  vars:
      apache2_log_level: "warn"

  handlers:
  - name: restart apache
    service:
      name: apache2
      state: restarted
      enabled: yes
    notify:
      - Wait for instances to listen on port 80
    become: yes

  - name: reload apache
    service:
      name: apache2
      state: reloaded
    notify:
      - Wait for instances to listen on port 80
    become: yes

  - name: Wait for instances to listen on port 80
    wait_for:
      state: started
      host: localhost
      port: 80
      timeout: 15
      delay: 5

  tasks:
  - name: Update cache
    apt:
      update_cache: yes
      cache_valid_time: 7200
    become: yes

  - name: Install packages
    apt:
      name={{ item }}
    loop:
      - apache2
      - logrotate
    notify:
      - restart apache
    become: yes

  - name: Configure apache2 log level
    lineinfile:
      path: /etc/apache2/apache2.conf
      line: "LogLevel {{ apache2_log_level }}"
      regexp: "^LogLevel"
    notify:
      - reload apache
    become: yes
...
```

### Explanation:
- `hosts` specifies the list of host or host groups against which we want to run the tasks in the play
- `vars` help to define variables to be used in the playbook. Ansible has 21 levels of variable precedence
- `handlers` are tasks that can be triggered (notified) during execution of a playbook, but they execute at the very end of a playbook.
- `tasks` are executed in order, one at a time, against all machines matched by the host pattern. The goal of each task is to execute a module, with very specific arguments
- `wait_for` conditional is to wait for some specific conditions to be matched before proceeding. The parameters mentioned in above playbook include:
    - `state`: ensure that the required port is open
    - `host`: hostname/IP address to wait for
    - `port`: port Number to poll
    - `timeout`: maximum number of seconds to wait for (default: 300)
    - `delay`: number of seconds to wait before starting to poll
- `loop`: Repeated tasks can be written as standard loops over a simple list of strings. Here, `loop` along with `{{ item }}` is used to install 2 packages using `apt`
- `become`: Ansible allows you to ‘become’ another user, different from the user that logged into the machine. Such escalation of privileges can be activated by setting `become` to `true` or `yes`.
- All parameters for the `apt` module can be found [here](https://docs.ansible.com/ansible/latest/modules/apt_repository_module.html)
- All parameters for the `lineinfile` module can be found [here](https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html)

1. Here, first `apt-cache` is updated
2. Next, the packages `apache2` and `logrotate` are installed via `apt` & once completed, a notification is send to `restart apache`
3. Now, the log level of apache2 is configured according to the variable set initially & a notification is sent to `reload apache`
4. These 2 handlers are triggered at the end of the playbook. Both of them notify `Wait for instances to listen on port 80`, but this is executed only once in the end (this is the way handlers are designed to work)

### Registering variables:

When a task is executed & the returned value is saved in a variable for later tasks, a __registered variable__ is created.

As an example:
```yaml
- hosts: web_servers

  tasks:

     - shell: /usr/bin/foo
       register: foo_result
       ignore_errors: True

     - shell: /usr/bin/bar
       when: foo_result.rc == 5
```

Here, the data structure returned by issuing of `/usr/bin/foo` is registered into a variable called `foo_result`. The `foo_result.rc` checks a __return value__ (here, _return code_) of the registered variable. Some other return values are:
- `changed` [boolean indicting if the task has changed the state of target server or not]
- `rc` [return code after using command line utilities]
- `msg` [generic message relayed to a user]
- `skipped` [whether the task was skipped or not]
- `stderr` [error output of command line utilities]
- `stdout` [normal output of command line utilities]

## Roles

A role in Ansible is the primary mechanism for breaking a playbook into multiple files. This simplifies writing complex playbooks, and it makes them easier to reuse. The breaking of playbook allows you to logically break the playbook into reusable components.

A role is a __group of variables, tasks, files, and handlers that are stored in a standardized file structure.__

* Roles are created using `$ ansible-galaxy init <role-name>`
* Roles are included in a play within a playbook by:
  ```yaml
  roles:
    - <role-1-name>
    - { role: <role-2-name>, some_variable: variable, tags: ["tag1", "tag2"] }
  ```

### Example scenario:
We need to configure all the `webservers` group in our inventories to have `apache2` installed, with a custom webpage (`index.html`) & custom LogLevel. We can do this using a playbook as shown above. But say, we want to use these same tasks as a subset of tasks in another playbook. Instead of writing code all over again, we create a role (say `apache-role`) & call this role in the necessary playbooks to execute the tasks.

```bash
# Create a new role
$ ansible-galaxy init apache-role

# Structure of the role directory
$ tree apache-role
apache-role/
├── defaults        # default variables for the role
│   └── main.yml
├── files           # files which can be deployed via this role
├── handlers        # handlers which may be used by this role
│   └── main.yml     
├── meta            # some meta data for this role
│   └── main.yml
├── README.md
├── tasks           # main list of tasks to be executed
│   └── main.yml
├── templates       # templates which can be deployed via this role
├── tests
│   ├── inventory
│   └── test.yml
└── vars            # other variables for the role 
    └── main.yml

# Create the desired index.html file in apache-role/files/
$ cat > apache-role/files/index.html
<fill-in-necessary-html-code>

# Create the apache2.conf.j2 template file by copying content from https://gist.github.com/TheSunMan/4008088 & replacing the LogLevel property with an Ansible variable
# The modification is shown by:
$ grep {{ apache-role/templates/apache2.conf.j2
LogLevel {{ apache2_loglevel }}

# Create this default variable
cat > apache-role/defaults/main.yml
---

apache2_loglevel: warn


# Create the tasks needed to achieve the objective. It is better to have tasks wit a common objective in a separate file & we `include` those files in the main.yml
$ cat > apache-role/tasks/apache2.yml
---

- name: install packages
  apt: name=apache2
  notify: restart apache handler

- name: set apache2.conf template
  template:
    src=apache2.conf.j2
    dest=/etc/apache2/apache2.conf

- name: set index template
  copy:
    src=index.html
    dest=/var/www/html/index.html

$ cat > apache-role/tasks/main.html
---

- include: apache2.yml

# Add the necessary handlers
$ cat > apache-role/handlers/main.yml
---

- name: restart apache handler
  service:
    name: apache2
    state: restarted
    enabled: yes
  notify:
    - Wait for instances to listen on port 80
  become: yes

- name: Wait for instances to listen on port 80
  wait_for:
    state: started
    host: localhost
    port: 80
    timeout: 15
    delay: 5

# Now, we create the playbook to deploy the role
$ cat > apache2-playbook.yml
---
- hosts: ubuntu-targets
  become: yes
  roles:
    - my_role

# Run the playbook to leverage the role
$ ansible-playbook apache2-playbook.yml
```