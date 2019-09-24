# Minishift - OKD
Minishift is the single node version of OpenShift container platform that you cna run locally. [Reference](https://docs.okd.io/latest/minishift/index.html)

# Install
https://docs.okd.io/latest/minishift/getting-started/index.html

# Getting started
1. Create a profile and install few addons before running Minishft
   - Decide on a profile name usually a project name. In this example I'm using the name "anair" and using Minishift to run in v3.11
    
            minishift profile set anair && minishift config set memory 4GB && minishift config set openshift-version v3.11.0 && minishift config set network-nameserver 8.8.8.8 && minishift addon enable anyuid && minishift addon enable admin-user
2. Start Minishift with Openshift v3.11
   
        minishift start --profile anair
        # Wait for few minutes till you get OpenShift server started.
        
        The server is accessible via web console at:
            https://IP address:8443
        
        You are logged in as:
            User:     developer
            Password: <any value>
        
        To login as administrator:
            oc login -u system:admin
        
        # Add OC client
        minishift oc-env && eval $(minishift oc-env)
        
        # Add minishift VM docker daemon access
        minishift docker-env && eval $(minishift docker-env) && docker ps
        
        # Login as admin through OC
        oc login -u admin

3. Login as admin/admin in the web ui
4. Create a Project and deploy a sample docker image

        # Create new project "anair-app"
        oc new-project anair-app
        
        # Login to minishift docker registry
        docker login -u $(oc whoami) -p $(oc whoami -t) $(minishift openshift registry)
        
        # Run the following commands that will run against minishift docker environment. Assuming a new zookeeper docker file is in your local
        docker build -t anair/zookeeper:3.4.10-1 .
        docker tag [TAG_NUMBER] 172.30.1.1:5000/anair-app/zookeeper
        
        # This will push the custom zookeeper image to "anair-app" openshift project image stream
        docker push 172.30.1.1:5000/anair-app/zookeeper
        
        # Verify image stream
        oc get istag
1. Stop minishift VM after use `minishift stop`