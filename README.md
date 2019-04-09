# Zero to JupyterHub with Kubernetes

## Main steps:
1. Setup Helm
2. Setup Jupyterhub

## Set up helm in your own namspace

1. Create a namespace for your jupyterhub
   
   ```
   Kubectl create namespace <your-namespace>
   ```

2. Download and install `Helm`:
   
   ```
   curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash
   ```
   

3. Create a `ServiceAccount` for `helm`'s `tiller` component to use.

   ```
   kubectl --namespace=<your-namespace> create serviceaccount tiller
   ```

4. Create a `RoleBinding` to give the `tiller` `ServiceAccount` enough
   permissions.

   ```
   kubectl --namespace=<your-namespace> apply -f tiller-rolebinding.yaml
   ```

5. Initialize `helm` and tiller

   ```
   helm --tiller-namespace=<your-namespace> init --service-account tiller
   ```

6. Wait for a few seconds, and test if `helm` is working.

   ```
   helm --tiller-namespace=<your-namespace> list
   ```

   This should return an empty response, rather than an error.

7. Ensure `tiller` is secure from access inside the cluster.
  
   ```
   kubectl --namespace=<your-namespace> \
           patch deployment tiller-deploy \
           --type=json \
           --patch='[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/tiller", "--listen=localhost:44134"]}]'
   ```

8. You can verify that you have the correct version and that it installed properly by running.
   
   ```
   helm --tiller-namespace=<your-namespace> version
   ```
Congratulations, Helm is now set up!

## Set up JupyterHub  
   
1. Add the Jupyterhub helm repository.

   ```
   helm --tiller-namespace=<your-namespace> repo add jupyterhub https://jupyterhub.github.io/helm-chart/
   helm --tiller-namespace=<your-namespace> repo update
   ```

   Note that you will need to add `--tiller-namespace=<your-namespace>` to all helm commands


2. Generate and add random bytes for proxy.

   ```
   openssl rand -hex 32
   ```
   
   Add output to `jupyterhub/config.yaml`:
  
   ```
   proxy:
     secretToken: "xxx"
   ```

3. Install the hub!

   ```
   #It is suggested to use your namaspace for both release and namespace
   RELEASE=<your-namespace>
   NAMESPACE=<your-namespace>
   helm --tiller-namespace=<your-namespace> upgrade --install $RELEASE jupyterhub/jupyterhub \
     --namespace=<your-namespace> \
     --version=0.8.0 \
     --values config.yaml
   ```

   You will need to modify `jupyterhub/config.yaml` to set the hostname
   to something other than the current default.

4. Wait for the pods to be ready, and enjoy your JupyterHub!

5. To use JupyterHub, enter the external IP for the proxy-public service in to a browser. 

6. To tear down everything:
   
   ```
   helm --tiller-namespace=<your-namespace> delete --purge <your-namespace>
   ```

