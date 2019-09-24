# Opendistro Elasticsearch
Deploy OpenDistro Elasticsearch cluster on commodity hardware

## Install
[Instructions](https://opendistro.github.io/for-elasticsearch-docs/docs/install/)

## Create SSL Certs
Following certs should be created through openssl for the nodes to communicate among each other and also for external components to talk to the ES cluster.
 - Root-ca cert and key  - only 1 time and shared by all nodes, Kibana and Fluentd
 - Admin cert and key - only 1 time and shared by all nodes
 - Node cert and key for each node
 - Create all certs in _/etc/elasticsearch_

### Run on only 1 node
Create root-ca.pem and admin.pem ina ny 1 node and copy the files to all other ES nodes

        # Root CA
        openssl genrsa -out root-ca-key.pem 2048
        # Cert valid for 3650 days
        # In the step below, make sure country, state, location, Org, OU are entered properly and remember those. Enter CN=elasticsearch
        openssl req -new -x509 -sha256 -key root-ca-key.pem -out root-ca.pem -days 3650
        # Validate expiry date
        openssl x509 -enddate -noout -in root-ca.pem
        
        
        # Admin cert
        openssl genrsa -out admin-key-temp.pem 2048
        openssl pkcs8 -inform PEM -outform PEM -in admin-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out admin-key.pem
        
        # In the CSR step below, make sure country, state, location, Org, OU are entered properly and remember those. Enter CN=Admin
        openssl req -new -key admin-key.pem -out admin.csr
        openssl x509 -req -in admin.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out admin.pem -days 3650
        # Validate expiry date
        openssl x509 -enddate -noout -in admin.pem
        
        rm admin-key-temp.pem
        # Copy CA cert and admin cert to other servers
        scp root-ca*.pem admin*.pem user@server:/tmp/escert

### Run on every node
Create Node level cert for each node individually

        # Node cert
        openssl genrsa -out node-key-temp.pem 2048
        openssl pkcs8 -inform PEM -outform PEM -in node-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out node-key.pem
        
        # In the CSR step below, make sure country, state, location, Org, OU are entered properly and remember those. Enter CN={hostname}
        openssl req -new -key node-key.pem -out node.csr
        openssl x509 -req -in node.csr -CA root-ca.pem -CAkey root-ca-key.pem -CAcreateserial -sha256 -out node.pem -days 3650
        # Validate expiry date
        openssl x509 -enddate -noout -in node.pem
        rm node-key-temp.pem
        # Copy certs to elasticsearch directory
        sudo cp root-ca*.pem /etc/elasticsearch && sudo cp node*.pem /etc/elasticsearch && sudo cp admin*.pem /etc/elasticsearch
        
        # Verify certs are copied and check date time
        sudo ls -ltr /etc/elasticsearch

### Cleanup
        rm /etc/elasticsearch/kirk.pem && rm /etc/elasticsearch/kirk-key.pem && rm /etc/elasticsearch/esnode.pem && rm /etc/elasticsearch/esnode-key.pem && rm /etc/elasticsearch/*-temp-key.pem

## Configuration
- Update JVM memory and args in _/etc/elasticsearch/jvm.options_
- Open _conf/elasticsearch.yml_
  - Replace all variables that have double curly braces with appropriate values
  - Decide on a cluster name
  - Decide the nodes that will be master nodes. Recommended to have 3 master and > 3 data nodes
  - Add node host names and assign and node name
  - Update admin_dn and node_dn with the values used to create the cert

## Start cluster
- After configuration is done, start master nodes 1 at a time by `sudo service elasticsearch start`
- Start data nodes
- Verify the cluster is formed and master node is elected through logs at `sudo tail -f /var/log/elasticsearch/{CLUSTER_NAME}.log`
-  
### Restart Cluster
- Restart cluster only if there is an error in ES processing or elasticsearch.yml is updated or if a nodes are updated: `sudo service elasticsearch restart`

# Security configuration
Elasticsearch can be secured through Okta and the default internal database. Users like admin, kibanaserver, fluentd and curator are created in the internal database. The custom _config.yml_ will enable Okta integration and use of the internal database uers.

## Okta integration
Okta integration is done through SAML 2.0. Work with the security team to enbale SAML integration.

- Refer [Opendistro SAML setup](https://opendistro.github.io/for-elasticsearch-docs/docs/security-configuration/saml/)
- Refer saml authc configuration in _conf/config.yml_
- Okta and basic auth integration is enabled in a chained implementation to allow for both internal/system users and browser users to use ES.

### Implement configuration 
- Login to each any ES node (do only in one node and not all nodes)
- Backup existing configuration
       
        cd /usr/share/elasticsearch/plugins/opendistro_security/securityconfig
        sudo cp config.yml config-old.yml


- Copy updated config.yml to _/usr/share/elasticsearch/plugins/opendistro_security/securityconfig_
- Run command to add saml auth to ES cluster in the _.opendistro_security_ index
  
        cd ../tools
        sudo ./securityadmin.sh -f ../securityconfig/config.yml -icl -nhnv -nrhn -cert /etc/elasticsearch/admin.pem -cacert /etc/elasticsearch/root-ca.pem -key /etc/elasticsearch/admin-key.pem -t config

> **DO NOT restart ES cluster for security config change**

## Internal user database
> Use internal roles for system integration only. 

Internal roles and users are available in _/usr/share/elasticsearch/plugins/opendistro_security/securityconfig_. Any changes in these files have to be set in the ES index _.opendistro_security_ using _/usr/share/elasticsearch/plugins/opendistro_security/tools/securityAdmin.sh_. These entries are also visible in Kibana. 

### Internal user files
Internal user files are at _/usr/share/elasticsearch/plugins/opendistro_security/securityconfig_:

- __internal_users.yml__: Add/update new reserved and internal users that is outside of Okta/AD/LDAP.
- __roles_mapping.yml__: Map role to users and/or Okta roles.
- __action_groups.yml__: Cluster and index permission groups here
- __roles.yml__: Hard-coded backend roles
- __tenants.yml__: Noop

### Systen users
- admin
- kibanaserver
- fluentd
- curator

### Kibana UI
- Add roles, users and role mapping in Kibana UI as an admin

### Update user/roles through internal files
#### Update user
- To change password or add a new user, create hash of the password `./usr/share/elasticsearch/plugins/opendistro_security/tools/hash.sh`
- When prompted, enter password
- Copy the hash that is generated and update in internal_users.yml for that user
- Make other changes to _internal_users.yml_ as required
- Apply the change to any ES node at: _/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/internal_users.yml_
- Run command to apply changes to _.opendistro_security_ index: 

        cd ../tools
        sudo ./securityadmin.sh -f ../securityconfig/internal_users.yml -icl -nhnv -nrhn -cert /etc/elasticsearch/admin.pem -cacert /etc/elasticsearch/root-ca.pem -key /etc/elasticsearch/admin-key.pem -t internalusers


#### Update role mapping
- To update role mapping, make changes to _roles_mapping.yml_
- Apply the change to any ES node at: _/usr/share/elasticsearch/plugins/opendistro_security/securityconfig/roles_mapping.yml_
- Run command to apply changes to _.opendistro_security_ index: 

        cd ../tools
        sudo ./securityadmin.sh -f ../securityconfig/roles_mapping.yml -icl -nhnv -nrhn -cert /etc/elasticsearch/admin.pem -cacert /etc/elasticsearch/root-ca.pem -key /etc/elasticsearch/admin-key.pem -t rolesmapping
