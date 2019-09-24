# Elasticsearch Curator
Deploy [Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/index.html) as the Elasticsearch maintenance tool. It supports deleting indices out of many other actions.

> Install and run Curator in all Master ES nodes

## Install
1. Skip pip install section if pip is already installed.
        
        # Install pip
        sudo curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
        sudo python get-pip.py
        
        # Verify install by printing pip version
        pip -V
        sudo rm get-pip.py

        # Install curator 
        sudo pip install elasticsearch-curator
        # If an error occurs due PyYAML, run the below command instead:
        sudo pip install elasticsearch-curator --ignore-installed PyYAML

        # Create curtor directory
        sudo mkdir /etc/curator
2. Copy conf files to the above directory

## Curator user and role
1. Login to Kibana as Admin
2. Create new user "curator"
3. Update "all_access" role mapping to add user "curator"

## Configuration
### curator.yml
This is the connection configuration file
1. Update current master node hostname
2. Optionally update cert path, if different
3. Provide above created user and password in "http_auth" in _userid:password_ format

### delete_indices.yml - Action file
- This file will have rules on deleting indices. Add additional index deletion actions in this file. Avoid creating new files.
- Application indices older than 30 days are deleted
- Kafka, Nifi and operational indices older than 30 days are deleted

## Run Curator
To perform a test, update delete_indices.yml file with a  index prefix and reduced age. Run dry run command to verify the indices list in __/var/log/curator.log__. Once verified, revert delete_indices.yml to the actual configuration.

    # Dry run - Print eligible indices in log
    sudo curator /etc/curator/delete_indices.yml --config /etc/curator/curator.yml --dry-run

    # Real run - Delete eligible indices
    sudo curator /etc/curator/delete_indices.yml --config /etc/curator/curator.yml
- Ignore warning "InsecureRequestWarning: Unverified HTTPS request is being made"
- Verify that intended indices are deleted from logs: __/var/log/curator.log__
- Run a GET query in Postman against another ES node to confirm index does not exist. Response code should be a 404.
    
        GET https://ES_HOSTNAME:9200/_cat/indices/{{DELETED_INDEX}}

### Schedule Run Curator
- Edit cron file `sudo crontab -e`
- Add schedule to delete indices at 7AM everyday
    
        PATH=/usr/lib64/qt-3.3/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin

        0 7 * * * curator /etc/curator/delete_indices.yml --config /etc/curator/curator.yml
