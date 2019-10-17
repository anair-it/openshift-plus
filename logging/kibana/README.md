# Opendistro Kibana
Deploy OpenDistro Kibana in an external cluster outside of Openshift. You may shutdown or delete the stock Kibana pods in Openshift

## Install
- Have a dedicated Kibana host/VM
- [Install instructions](https://opendistro.github.io/for-elasticsearch-docs/docs/kibana/)

## Security configuration
1. `sudo mkdir /etc/kibana/cert`
1. Copy ES root-ca.pem to _/etc/kibana/cert_

### Create internal certificate and key using openssl

    cd /etc/kibana/cert
    
    
    # Cleanup existing certs
    sudo rm kibana*
    
    
    # Root CA
    sudo openssl genrsa -out kibana-ca-key.pem 2048
    # Cert valid for 3650 days
    # In the step below, make sure country, state, location, Org, OU are entered properly and remember those. Enter CN=kibana
    # Do not enter email id and server name. Leave everything else blank.
    sudo openssl req -new -x509 -sha256 -key kibana-ca-key.pem -out kibana-ca.pem -days 3650
    # Validate expiry date
    sudo openssl x509 -enddate -noout -in kibana-ca.pem
    
    
    # Node cert
    sudo openssl genrsa -out kibana-key-temp.pem 2048
    sudo openssl pkcs8 -inform PEM -outform PEM -in kibana-key-temp.pem -topk8 -nocrypt -v1 PBE-SHA1-3DES -out kibana-key.pem
    
    # In the CSR step below, make sure country, state, location, Org, OU are entered properly and remember those. Enter CN={hostname}
    # Do not enter email id and server name. Leave everything else blank.
    sudo openssl req -new -key kibana-key.pem -out kibana.csr
    sudo openssl x509 -req -in kibana.csr -CA kibana-ca.pem -CAkey kibana-ca-key.pem -CAcreateserial -sha256 -out kibana.pem
    
    sudo rm kibana-key-temp.pem

## Configuration
1. Enable https in kibana.yml
2. Add server certificate and key info created in above step
3. Enable SAML integration with Okta in ES config.yml and kibana.yml
4. Restart service `sudo service kibana restart`
5. Login to Kibana and verify changes

## Roles
- Work with the security team to create 2 roles in the security application like Okta.
- Provide a list of users with matching roles to security team

### Admin
- Okta role: KibanaAdmin
- Full access (God rights) to the ES cluster and indices
- Mapped to "all_access" role in Kibana Role Mappings

### User
- Okta role: KibanaUser
- KibanaUser: Readonly access to ES indices, create and update dashboards and visualizations
- Mapped to "kibana_user" and "readall" roles in Kibana Role Mappings

## Switch authentication
- Switch authentication from SAML to basic auth by uncommenting the properties in the "Basic auth" section. Comment the SAML properties

## Healthcheck
- Monitor CPU, memory through an external monitroing system like Nagios, Zabbix
- Healthcheck url: `curl -ksuI monitor:password https://KIBANA:5601 | grep -c 'HTTP/1.1 302 Found'`. 
  - If response is 0, Kibana is unhealthy and create an alert.
  - Update the hostname, userid and password as required