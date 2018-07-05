# Apigee OPDK Ansible

Centralize Edge operation tasks using one inventory file as reference for all tasks.


# Table of contents
1. [Requirements](#requirements)
1. [Getting ready](#getting-ready)
    1. [Configuration file](#config-file)
    1. [Topology definition](#topology-defintion)
    1. [Generate Anisble inventory](#inventory)
    1. [Generate response files](#gen-response)
1. [Apigee Edge Ops Tasks](#ops-tasks)
    1. [Install prereqs and the apigee-setup utility](#prereqs)
    1. [Port checking report](#port-check)
    1. [Install Edge](#install)
    1. [Update Edge](#update)
    1. [Edge onboarding](#onboarding)
    1. [Create keystore in Edge and upload keystore certs](#keystore)
    1. [Create a VirtualHost](#vhost)
    1. [Run apigee-service commands accross planet](#apigee-service)
    1. [Create Edge custom role](#custom-role)
    1. [Create user](#user)
    1. [Fetch planet logs](#logs)
    1. [Run planet scan](#planet-scan)
    1. [Ansible Ad-Hoc commands](#ad-hoc)
1. [Author](#author)


## Requirements <a name="requirements"/>



- ansible >= 2.2

## Getting ready <a name="getting-ready" />
### Configuration file <a name="config-file" />
Under group_vars/all directory there's an _env.yml_, fill the required variables. 

Property | Example | Description
--- | --- | ---
license_path* | /home/gump/license.txt |  License file absolute path including license file name.
apigee_user* | bubbagump | software.apigee.com user.
apigee_pwd* | mygumppassword | software.apigee.com password.
ssh_user | mgump | ssh user.
ssh_key | /home/gump/.ssh/gumptron.key | ssh key absolute path.
ssh_pwd | gumpsecret | ssh password. If not needed, leave the original value.
ssh_bastion_host | 35.0.0.3 | ssh bastion/jumpbox host. If not needed, leave the original value.
ssh_bastion_user | runforest | ssh bastion/jumbox user. If not needed, leave the original value.
ssh_bastion_key | /home/gump/runforest.key | ssh bastion/jumpbox key absolute path. If not needed, leave the original value.
cass_auth | y | Cassandra Authentication. If not needed set this to "n" else set this to "y"
pg_ram_in_mb* | 4096 | Postgresql machine memory required to set memory adjustments.
edge_version* | 4.18.01 |  Edge version. At this point is 4.18.01.
opdk_ldap_password | secret | Apigee LDAP password. Set this value else remove from the file so that it picks the default value.
opdk_admin_email** | forest@gump.com |  Edge installation admin email.
opdk_admin_password** | forestpwd | Edge installation admin password.
opdk_smtp_mail_from** | Admin OPDK | SMTP from.
opdk_smtp_mail_from** | opdk@example.com | Email Address of the SMTP From.
opdk_smtp_skip** | n | "y" to skip SMTP configuration or "n".
opdk_smtp_user** | forest@user.com | SMTP user.
opdk_smtp_password** | forestpwd | SMTP password.
opdk_smtp_host** | smtp.gump.com | SMTP host.
opdk_smtp_port** | 25 | SMTP port.
opdk_smtp_ssl** | n | "y" or "n".
onboard_org_name*** | BubbaGump | organization name for onboarding. If not needed, leave the original value.
onboard_new_user | n | Set value to "n" for not setting up a new user as Org Admin else set value to "y".
onboard_admin_username*** | forest@gump.com | org admin username (email format) for onboarding. 
onboard_admin_name*** | Forest | org admin name for onboarding. If not needed, leave the original value. 
onboard_admin_lastname*** | Gump | org admin lastname for onboarding. If not needed, leave the original value. 
onboard_admin_pwd*** | forestpwd | org admin password for onboarding. If not needed, leave the original value. 
onboard_env*** | test | environment name for onboarding. If not needed, leave the original value. 
onboard_vhost_alias*** | api.forest.com  | virtualhost alias for onboarding. If not needed, leave the original value.
vhost_port | 9001 | Port for Virtual host during Org Setup. Change if required, if not needed leave the original value.
vhost_name | default | Name of the Virtual host. Change if required, if not needed leave the original value.
vhost_ssl | n | SSL for Virtual host. If not required, leave the default value.
pg_name* | devportal |Specify the name of portal database in postgres.
pg_user*| apigee | postgres username.
pg_pwd* | postgres | postgres password.
drupal_pg_user* | drupaladmin | drupal admin.
drupal_pg_pass* | portalsecret |drupal admin password.
default_db* | postgres.
devportal_admin_firstname* | firstname | portal admin firstname.
devportal_admin_lastname* | lastname | portal admin lastname.
devportal_admin_username* | username | portal admin username.
devportal_admin_pwd* | password | password of portal admin.
devportal_admin_email* | user@example.com | email of portal admin to login into dev portal.
edge_org* | organization | org name to be connected to. 
mgmt_url* | management API | management URL of org.
devadmin_user* | username | org admin username to connect to org.
devadmin_pwd* | password | org admin password.
php_fpm_port* | 8888. 
keystore_jar | /tmp/keystore.jar | Location of the JAR file on the target server.
keystore_name | keystore | Name of the Keystore to be displayed in the Env Configuration.
keystore_alias | keystore | Name of the Keystore alias to be displayed in the Env Configuration.
keystore_password | secret | Password of the Key used for Authentication.
enable_system_check | n | Checking the System resources as part of the Setup/upgrade process. If not, required leave the default value.
apigee_mirror_type | none | Use "tar" as the value if you are using tar as the Apigee repo. Use "nginx" if you are using the nginx method. Use "none" if you are using software.apigee.com to pull the Apigee rpm's.
apigee_mirror_protocol | http | Use this if you are using nginx as the Mirror type and possible values are "http" or "https". If not, leave the default value.
apigee_mirror_url | mirror.example.com | If you are using the nginx method as the mirror type provide the url for the repo. If not, leave the default value.
apigee_mirror_port | 3939 | If you are using the nginx method as the mirror type provide the port for the repo. If not, leave the default value.
apigee_mirror_username | admin | If you are using the nginx method as the mirror type provide the user for the repo. If not, leave the default value.
apigee_mirror_pwd | admin | If you are using the nginx method as the mirror type provide the password for the repo. If not, leave the default value.

\* Required.

** Required if _opdk_smtp_skip=n_.

*** Required for onboarding.

**Note:** 
- If you are choosing **tar** are your apigee mirror type, place your apigee tar file in **apigee-repo** directory for the script to copy them into the remote nodes.

### Topology definition <a name="topology-defintion"/>

First create an edge topology definition json file (example _examples/topology-1dc-5n.json_).
This is the only file that you need to create.

Rename this file with the name that you think will best differentiates between rest of your planets.

Important variables that need to be changed:
  - **planet**: This value is the naming that you use to differentiate your planets 
  - **ip**: This is the DNS or the IP address that you use for SSH
  - **dns_or_ip**: This value is what gets resolved when we use hostname -i on that specific node.

### Generate Anisble inventory <a name="inventory"/>

Create an inventory file used by all Ansible playbooks and operation tasks:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" inventory.yml
```

Variable | Example | Description
--- | --- | ---
topolgy_src* | examples/topology.json | Path to topology json file.


\* Required

Find the files under:
  - _inventory/inventory\_PLANET.INI_
  
  ### Generate Anisble inventory for Dev Portal <a name="inventory"/>

Create an inventory file used by all Ansible playbooks and operation tasks:
```
$ ansible-playbook -e "topology_src=PATH_TO_TOPOLOGY_FILE" inventory.yml
```

Variable | Example | Description
--- | --- | ---
topolgy_src* | topology/devportal.json | Path to topology json file.


\* Required

Find the files under:
  - _inventory/

### Generate response files <a name="gen-response" />

Generate response file per region used by the installation, upgrade and other ops tasks. Also
create the onboarding response file:
```
$ ansible-playbook -i inventory/INVENTORY_FILE response_files.yml
```
Find the response files under:
  - _reports/PLANET/response_files/response_PLANET_REGION.cfg_
  
### Generate response files for Dev Portal <a name="gen-response" />

Generate response file per region used by the installation:
```
$ ansible-playbook -i inventory/INVENTORY_FILE response_files.yml --tags devportal
```
Find the response files under:
  - _reports/PLANET/response_files/responsefile.cfg_

## Apigee Edge Ops Tasks <a name="ops-tasks" />

### Install prereqs and the apigee-setup utility <a name="prereqs" />
Install the prerequisites described [here](https://docs.apigee.com/private-cloud/latest/install-edge-apigee-setup-utility) and some more
useful tools.

Installs the following packages across the planet:
- wget
- curl
- telnet
- nc  
- nmap  
- java-1.8.0-openjdk-devel
- apigee-service
- apigee-setup
- apigee-provision in management nodes

Uploads the license file, the response and onboarding files. 

Sets Cassandra, Message Processor and Postgresql memory settings.

```
$ ansible-playbook -i inventory/INVENTORY_FILE prerequisites.yml
```
### Install prereqs and the apigee-setup utility on Dev Portal node<a name="prereqs" />

```
$ ansible-playbook -i inventory/INVENTORY_FILE devreqs.yml
```

### Port checking report <a name="port-check" />

Create port report for a whole planet. This will test ports between nodes and create two CSV files:
- Connectivity report grouped by edge component.
- Compact report grouped by host.
```
$ ansible-playbook -i inventory/INVENTORY_FILE port_report.yml
```
Find the report files under: 
  - _reports/port_connectivity_report\_PLANET.csv_
  - _reports/port_compact\_PLANET.csv_

**Note:** The hosts require _nmap_, it is installed in the above prerequisites playbook.

### Install Edge <a name="install" />
Install Edge components in the planet:

```
$ ansible-playbook -i inventory/INVENTORY_FILE -e "cmd=setup" setup.yml
```

### Install Dev Portal <a name="install" />
Install Dev Portal:

```
$ ansible-playbook -i inventory/INVENTORY_FILE  dev_portal_setup.yml
```

### Update Edge <a name="update" />
Update Edge components in the planet, only if current version >= 4.17.01.

This update process is used for updating to 4.18.01, as part of the update process it supports Installing a new Postgres Standby with the Older version and then the upgrade process on rest of the nodes in the planet.

***Steps:***

- Edit the edge topology definition json as explained in [topology definition](#topology-defintion). Change the values of the existing Postgres Standby node with new standby node values and rename the file as shown below

```
# filename: examples/topology-1dc-5n.json
...

"planet": "UAT-1DC-5N"

...

...
            "ip": "old_node_ip_or_dns",
              "dns_or_ip": "old_node_ip_or_dns",
            "components": [{
                "comp": "PS"
              },
              {
                "comp": "QS"
              },
              {
                "comp": "QD"
              },
              {
                "comp": "PGs"
              }
            ]
            
...
```

```
# filename: examples/topology-1dc-5n_Standby.json
...

"planet": "UAT-1DC-5N_Standby"
...
...

            "ip": "new_node_ip_or_dns",
              "dns_or_ip": "new_node_ip_or_dns",
            "components": [{
                "comp": "PS"
              },
              {
                "comp": "QS"
              },
              {
                "comp": "QD"
              },
              {
                "comp": "PGs"
              }
            ]
            
...

```

- Now generate the inventory file as shown in [Generation of Ansible inventory](#inventory)
- Now run the below command that is going generate the response files as explained in [generation of response files](#gen-response)
- On the Postgres master node set the replication so that the PG master is aware of the new Standby node as described [here](https://docs.apigee.com/private-cloud/v4.18.01/update-apigee-edge-4170x-41801#requiredupgradetopostgres96-installinganewpostgresstandbynode)
    - On the PG Master edit the ***/opt/apigee/customer/application/postgresql.properties*** as below
    
    ```
    conf_pg_hba_replication.connection=host replication apigee existing_slave_ip/32 trust\ \nhost replication apigee new_slave_ip/32 trust
    ```
    
    - Restart the apigee-postgresql on the PG Master Node using the below command
    
    ```
    /opt/apigee/apigee-service/bin/apigee-service apigee-postgresql restart
    
    ```
    
- Run the below command that is going to install the required prereqs and setup of the new standby node.
```
$ ansible-playbook -i inventory/INVENTORY_FILE setup_new_pgstandby.yml
```
- After the new standby is installed use the actual topology JSON file ***examples/topology-1dc-5n.json*** and its corresponding inventory file (inventory/inventory\_PLANET.INI) for the upgrade process.
- Edit the file with below changes shown as an example considering a 5 node topology. Find for the lines accordingly for adding the new PG Standby node details in the inventory file.

```
...
#Planet UAT-1DC-5N
localhost ansible_connection=local
dc-1-n6 ansible_host=10.5.0.6 env_ref=DC1N6 region=dc-1 dns_or_ip=10.5.0.6

[planet]
#dc-1
dc-1-n1 ansible_host=10.5.0.1 env_ref=DC1N1 region=dc-1 dns_or_ip=10.5.0.1
dc-1-n2 ansible_host=10.5.0.2 env_ref=DC1N2 region=dc-1 dns_or_ip=10.5.0.2
dc-1-n3 ansible_host=10.5.0.3 env_ref=DC1N3 region=dc-1 dns_or_ip=10.5.0.3
dc-1-n4 ansible_host=10.5.0.4 env_ref=DC1N4 region=dc-1 dns_or_ip=10.5.0.4
dc-1-n5 ansible_host=10.5.0.5 env_ref=DC1N5 region=dc-1 dns_or_ip=10.5.0.5

...

...
[dc-1-pgstandby]
#Listing of all the Postgres standby nodes in dc-1
dc-1-n5 pg_conf=standby

[dc-1-pgNewStandby]
#Listing the new Postgres standby node in dc-1
dc-1-n6

[dc-1-ui]
#Listing of all the UI nodes in dc-1
dc-1-n1

...
```
- Set the value of the target version in your _env.yml_:
```
# filename: env.yml
...
edge_version: 4.18.01
...
```
And run:

```
$ ansible-playbook -i inventory/INVENTORY_FILE update.yml
```

### Edge onboarding <a name="onboarding" />
Onboarding: create organization, environment, org admin user and default virtual host.

```
$ ansible-playbook -i inventory/INVENTORY_FILE onboard.yml
```

### Create keystore in Edge and upload keystore certs <a name="keystore" />
[Apigee Keystores and Truststores](https://docs.apigee.com/api-services/content/keystores-and-truststores).

Create a jar Keystore from key/cert pair, an Apigee Keystore in Edge and upload the JAR to Edge:

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "keyalias=KEY_ALIAS keystore=KEYSTORE_NAME \
  ks_cert=PATH_TO_CERT ks_key=PATH_TO_KEY \
  ks_org=EDGE_ORG ks_env=EDGE_ENV" \
  keystore.yml
```

Variable | Example | Description
--- | --- | ---
keyalias* | my_key_alias | Key alias. 
keystore* | keystore | Keystore name.
ks_cert* | tls/server.crt  | Path to cert file.
ks_key* | tls/server.key | Path to key file.
ks_org* | BubbaGump | Edge Org where the Keystore is going to be created.
ks_env* | test | Edge Env where the Keystore is going to be created.

\* Required.

### Create a VirtualHost <a name="vhost"/>
Create virtual host in an existing Edge Org/Env. Optionally TLS could be enabled with an existing keystore.

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "keyalias=KEY_ALIAS keystore=KEYSTORE_NAME \
  vhost_name=VHOST_NAME vhost_aliases=COMMA_SEPARATED_ALIASES \
  org=EDGE_ORG env=EDGE_ENV tls_enabled=true" \
  vhost_tls.yml
```

Variable | Example | Description
--- | --- | ---
vhost_name* | tls/server.crt  | Path to cert file.
vhost_aliases* | tls/server.key | Path to key file.
org* | BubbaGump | Edge Org where the Keystore is going to be created.
env* | test | Edge Env where the Keystore is going to be created.
tls_enabled* | true | Enable TLS. 
keyalias** | my_key_alias | Existing key alias. 
keystore** | keystore | Existing keystore name.

\* Required.

** Required if tls_enabled is set to any value.

### Run apigee-service commands accross planet <a name="apigee-service" />

Run an apigee-service command in particular components or across the planet.

```
$ ansible-playbook -i inventory/INVENTORY_FILE -e "cmd=COMMAND component=COMPONENT" setup.yml
```

Values for **cmd**:
- status
- start
- wait_for_ready
- stop
- restart

Values for **component**:
- zk      (Zookeeper)
- cs      (Cassandra)
- ds      (Zookeeper and Cassandra)
- ldap    (OpenLDAP)
- ms      (Management Server)
- msldap  (Management Server and OpenLDAP)
- r       (Router)
- mp      (Message Processor)
- rmp     (Router and Message Processor)
- qs      (QPIDD and Qpid Server)
- pg      (Postgreql and Postgres Server)
- all     (All Edge components)

### Create Edge custom role <a name="custom-role">
[Edge Roles](https://docs.apigee.com/api-services/content/managing-roles-api).

Create custom role in an Edge Org.

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "api_action=roles org=ORG role=ROLE_NAME \
  customrole_path=ROLE_PATH" adminapi.yml
```

Variable | Example | Description
--- | --- | ---
api_action* | roles | This must be _roles_ to create new roles.
org* | BubbaGump | Edge Org where the role is going to be created.
role* | newrol | Role name.
customrole_path* | examples/customrole.json | Path to role in JSON format. 

\* Required.

Each Edge Org has some [built-in roles](https://docs.apigee.com/api-services/content/edge-built-roles):
- **Organization Administrator**: Super user. Has full CRUD access to resources in the organization. In an Edge 
for Private Cloud installation, the most powerful role is the System administrator role, which also has access 
to system-level functions that the Organization Administrator doesn't.
- **Operations Administrator**: Deploys and tests APIs; has read-only access to other resources.
- **Business User**: Creates and manages API products, developers, developer apps, and companies; 
creates custom reports on API usage; has read-only access to other resources.
- **User**: Creates API proxies and tests them in the test environment; has read-only access to other resources.

### Create user <a name="user" />
[Edge Users](https://docs.apigee.com/private-cloud/latest/managing-users-roles-and-permissions).

Create user and add to an existing role.

```
$ ansible-playbook -i inventory/INVENTORY_FILE \
  -e "api_action=users org=ORG role=ROLE_NAME \
  user_email=USER_EMAIL user_password=USER_PASSWORD \
  user_name=USER_NAME user_lastname=USER_LASTNAME" \
  adminapi.yml
```

Variable | Example | Description
--- | --- | ---
api_action* | users | This must be _users_ to create new user.
org* | BubbaGump | Edge Org where the role is going to be created.
role | User | Edge role.
user_email* | forest@gump.com | User email (used as _username_).
user_password* | mypassword | User password. 
user_name* | Forest | User name. 
user_lastname* | Gump | User last name.

\* Required.

### Fetch planet logs <a name="logs" />
Tar all the edge logs from each node.

```
$ ansible-playbook -i inventory/INVENTORY_FILE logs.yml
```

### Run planet scan <a name="planet-scan"/>
Run check commands across the planet:

- _Edge pods_: List servers for each Pod.
    - Central pod.
    - Gateway pod.
    - Analytics pod.

- _Zookeeper_: Run Zookeeper status commands.
    - _ruok_.
    - _stat_.
    - Zookeeper tree.

- _Cassandra_: Run Cassandra nodetool commands.
    - ring.
    - status.
    - statusthrift

- _Postgres_: Check master/standby nodes.

- _Management ports check (self status)_: Get info from each Edge component.
    - Management Server (8080).
    - Router (8081).
    - Message Processor (8082).
    - Qpid Server (8083).
    - Postgres Server (8084).

- _Analytics groups_: List the Analytics groups and servers.

- Fetch _customer/application_ files across the planet.

- Memory and disk usage.

```
$ ansible-playbook -i inventory/INVENTORY_FILE planet_scan.yml
```

Find the scan files:
  - _reports/scan_:

### Ansible Ad-Hoc commands <a name="ad-hoc" />
[Ansible Ad-Hoc commands](http://docs.ansible.com/ansible/latest/intro_adhoc.html)

Example: 
```
$ ansible -i inventory/INVENTORY_FILE HOST_GROUP -m shell -a '/opt/apigee/apigee-service/bin/apigee-all status'
```
Script for inventory group hosts:
Run _echo "Hello node!"_ command example:
```
$ ./service.sh -i inventory/INVENTORY_FILE -g HOST_GROUP -c "echo 'Hello node!'"
```
Where:
- **-h**: help
- **-i**: ansible inventory path. **Required**
- **-c**: command to run in the node.
- **-g**: ansible inventory group

Default values:
- If not _-c_ option provided, the default command is: _/opt/apigee/apigee-service/bin/apigee-all status_
- If not _-g_ option provided, the default group is: _planet_
```
$ ./service.sh -i inventory/INVENTORY_FILE -g HOST_GROUP -c "echo 'Hello node!'"
```

\* _/opt/apigee/apigee-service/bin/apigee-all start|restart_ are not recommended since the start order is important: [Starting apigee components](https://docs.apigee.com/private-cloud/latest/starting-stopping-and-restarting-apigee-edge)

## Author <a name="author" />

If you have any questions regarding this project contact:  
Srikanth Mushty <srikanthm@sidgs.com>
