
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
                  --path=./logging \
                  --personal


Install Elasticsearch using official helm chart and Vector agent (https://vector.dev/) through official helmchart
Also install kibana on the K8s cluster using helm chart.
Configure vector to input http requests and log the http requests to Elasticsearch. The same should be visualized in kibana.
Use a test client to simulate traffic.
All installation and configuration must be done via Gitops using FluxCD.


