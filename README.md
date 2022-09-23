
Task 1 - Install a multi node k8s cluster in your laptop, use any opensource distribution to set this up.
        - Local Environment - Ubuntu on VirtualBox
        
        - Installed Docker:
            $curl -fsSL https://test.docker.com -o test-docker.sh
            $sudo sh test-docker.sh
            
        - Installed Kubectl to interact with the cluster:
            $curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
            chmod +x kubectl
            mkdir -p ~/.local/bin
            mv ./kubectl ~/.local/bin/kubectl
            export PATH="$HOME/.local/bin:$PATH"

        - Used Kind to spin up a multi-node cluster named "kind-gitops", having 1 control-plane and 2 worker nodes
            - Installation of KinD:
                curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.15.0/kind-linux-amd64
                chmod +x ./kind
                sudo mv ./kind /usr/local/bin/kind 
            - To create cluster we make use of a config file named "kind-config.yaml" which has:
                  kind: Cluster
                  apiVersion: kind.x-k8s.io/v1alpha4
                  nodes:
                  - role: control-plane
                  - role: worker
                  - role: worker
             - Run command "kind create cluster --name gitops --config kind-config.yaml
             
             
Task 2 - Use FluxCD tool (https://fluxcd.io/) to setup installations in the cluster:

           - Installed the Flux CLI
             $curl -s https://fluxcd.io/install.sh | sudo bash
             
           - Created a Github Personal Access Token
           
           - Exported the token and username:
               export GITHUB_TOKEN=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
               export GITHUB_USER=syedabrarali
               
           - Installed Flux:
                flux bootstrap github \
                  --owner=$GITHUB_USER \
                  --repository=fluxCD_K8s \
                  --branch=main \
                  --path=./cluster/my-cluster/ \
                  --personal


Task 3 - Install Elasticsearch using official helm chart and Vector agent (https://vector.dev/) through official helmchart

  - Cloned the fluxCD_K8s repo.
  - Created a namespace named “elastic-search”
  - Using flux created a GitRepository Object to obtain the elastic search Helm chart:

       
  - Imperative command to create gitrepository object
        ```
        flux create source git elastic-search --url=https://github.com/elastic/helm-charts \
        --branch=main \
        --interval=30s \
        --namespace=elastic-search \
        --export > ./cluster/my-cluster/elastic-search/es-source.yaml
        ```

   - The manifest generated is below:
        ```
        **es-source.yaml**

        ---
        apiVersion: source.toolkit.fluxcd.io/v1beta2
        kind: GitRepository
        metadata:
          name: elastic-search
          namespace: elastic-search
        spec:
          interval: 30s
          ref:
            branch: main
          url: https://github.com/elastic/helm-charts
        ```

   - Commited the changes to the fluxCD_K8s repo
   - Created a helmrelease object and stored it in es-helmrelease.yaml file
   
   - Imperative command to create helmrelease object:
        ```
        flux create helmrelease elastic-search --chart elasticsearch \
        --source GitRepository/elastic-search --namespace elastic-search \
        --export > ./cluster/my-cluster/elastic-search/es-helmrelease.yaml
        ```
    - The manifest generated is below:
        ```
        **es-helmrelease.yaml

        ---
        apiVersion: helm.toolkit.fluxcd.io/v2beta1
        kind: HelmRelease
        metadata:
          name: elastic-search
          namespace: elastic-search
        spec:
          chart:
            spec:
              chart: elasticsearch
              reconcileStrategy: ChartVersion
              sourceRef:
                kind: GitRepository
                name: elastic-search
          interval: 1m0s**
```

         - commited this to the fluxCD_K8s repo
         - The elasticsearch is up and running in namespace elastic-search.
         - Port forwarded the elasticsearch-master service to 9200

                ```
                kubectl -n elastic-search port-forward svc/elasticsearch 9200
                ```

         - Same Procedure was followed for deploying vector from helm-chart obtained from https://github.com/vectordotdev/helm-charts/charts/vector. 
         - Created a gitrepository for vector in new vector namespace named it vector1 and created a helmrelease object called vector.
         
Task 4 - Also install kibana on the K8s cluster using helm chart:

          - Cloned the fluxCD_K8s repo.
          - Created a namespace named “kibana”
          - Using flux created a GitRepository Object to obtain the kibana Helm chart:


        - Imperative command to create gitrepository object:
        ```
        **flux create source git kibana --url=https://github.com/elastic/helm-charts \
        --branch=main \
        --interval=30s \
        --namespace=elastic-search \
        --export > ./cluster/my-cluster/kibana/kb-source.yaml**
        ```

       - The manifest generated is below:
              ```
                **kb-source.yaml**

                **---
                apiVersion: source.toolkit.fluxcd.io/v1beta2
                kind: GitRepository
                metadata:
                  name: kibana
                  namespace: kibana
                spec:
                  interval: 30s
                  ref:
                    branch: main
                  url: https://github.com/elastic/helm-charts**

                ```

          - Commited the changes to the fluxCD_K8s repo
          - Created a helmrelease object and stored it in kb-helmrelease.yaml file
          
        - Imperative command to create gitrepository object:
        ```
        flux create helmrelease kibana --chart kibana \
        --source GitRepository/kibana --namespace kibana \
        --export > ./cluster/my-cluster/kibana/kb-helmrelease.yaml
        ```
        
        - The manifest generated is below:
        ```
        **kb-helmrelease.yaml

        ---
        apiVersion: helm.toolkit.fluxcd.io/v2beta1
        kind: HelmRelease
        metadata:
          name: kibana
          namespace: kibana
        spec:
          chart:
            spec:
              chart: kibana
              reconcileStrategy: ChartVersion
              sourceRef:
                kind: GitRepository
                name: kibana
          interval: 1m0s**
        ```

          - commited this to the fluxCD_K8s repo
          - Kibana is up and running in namespace Kibana.
          - Port forwarded the kibana deployment to 5601:
                $kubectl port-forward deployment/kibana-kibana 5601

Task 5 - Configure vector to input http requests and log the http requests to Elasticsearch. The same should be visualized in kibana.
          
          - Tried to configure a Apache Web server on an EC2 instance and direct that as the source for vector and Elasticsearch as the sink for vector in the                     vector.toml config file but wasn't able to make it work.
          - Below is the sample config file I created:
                [sources.apache]
                type = "http"
                address = "13.127.201.123:80"  #address of the httpd server
                encoding = "text"
                headers = [ "User-Agent" ]
                
                [sinks.elasticsearch]
                type = "elasticsearch"
                inputs = [ "apache" ]
                endpoint = "http://10.96.240.135:9200"
                mode = "bulk"
                compression = "none"

        
Task 6 - Use a test client to simulate traffic.
        
        - Created a HTTP server on ec2 instance - 13.127.201.123:80
        - curl request to generate traffic so that vector can collect the request data and pass it on to elasticsearch as the sink.
        
Task 7 - All installation and configuration must be done via Gitops using FluxCD.
        - All installations were done via FluxCD using the GitOps methodology by keeping fluxCD_K8s git repo as the single source of truth.


