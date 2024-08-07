 



Kubernetes Cluster
==================

Kubernetes is a portable, extensible, open source orchestration platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. CloudFabrix's RDA supports integrating with Kubernetes cluster through it's CLI interface (**kubectl**) over SSH or **HTTP API** interface to collect the inventory of Kubernetes resources such as pods, nodes, services, deployments etc.. on-demand or on schedule basis.

****1\. Prerequisites:****
--------------------------

Create a service user account with SSH login privileges to connect to one of the Kubernetes master node (on which **kubectl** was installed) to collect the inventory data. The user should have privileges to run the below command.

*   kubectl get \*

Info

If kubectl is enabled only for root user, provide sudo privileges to service user account that was created in the above step.

****2\. Inventory collection using kubectl over SSH:****
--------------------------------------------------------

On CentOS, RHEL & Ubuntu, edit **/etc/sudoers** file and add the **kubetctl** command for the user account that is used for data collection. Below configuration setting restricts the service user account to execute only the **kubectl get** \* commands and does not allow any other administrative commands.

`[](#__codelineno-0-1) user_name ALL=(root) NOPASSWD: /usr/bin/kubectl get *`

Info

SSH user authentication supports both password and SSH key.

****3\. Inventory collection using HTTP API:****
------------------------------------------------

Follow the below steps to create a service user account with read-only permissions to Kubernetes resources for inventory data collection using Kubernetes HTTP API interface.

**Step-1**: Create a Namespace for service user account

`[](#__codelineno-1-1) kubectl create namespace cfx-kube-discovery`

**Step-2**: Create a service user account and associate it to the above namespace.

`[](#__codelineno-2-1) kubectl create serviceaccount cfx-api-access -n cfx-kube-discovery`

**Step-3**: Create a a role for service user account with below configuration contents and apply the role configuration.

`[](#__codelineno-3-1) cat > cfx-api-access-role.yml << 'EOF' [](#__codelineno-3-2) apiVersion: rbac.authorization.k8s.io/v1 [](#__codelineno-3-3) kind: ClusterRole [](#__codelineno-3-4) metadata: [](#__codelineno-3-5)   name: cfx-api-access-role [](#__codelineno-3-6)   namespace: cfx-kube-discovery [](#__codelineno-3-7) rules: [](#__codelineno-3-8)   - apiGroups: [""] [](#__codelineno-3-9)     resources: ["*"] [](#__codelineno-3-10)     verbs: ["get", "list", "watch"] [](#__codelineno-3-11)   - apiGroups: ["apps"] [](#__codelineno-3-12)     resources: ["*"] [](#__codelineno-3-13)     verbs: ["get", "list", "watch"] [](#__codelineno-3-14) [](#__codelineno-3-15) EOF`

`[](#__codelineno-4-1) kubectl apply -f cfx-api-access-role.yml`

**Step-4**: Create a role binding for the service user account with the below configuration contents and apply the role bindings configuration.

`[](#__codelineno-5-1) cat > cfx-api-access-role-binding.yml << 'EOF' [](#__codelineno-5-2) apiVersion: rbac.authorization.k8s.io/v1 [](#__codelineno-5-3) kind: ClusterRoleBinding [](#__codelineno-5-4) metadata: [](#__codelineno-5-5)   name: cfx-api-access-role-binding [](#__codelineno-5-6)   namespace: cfx-kube-discovery [](#__codelineno-5-7) subjects: [](#__codelineno-5-8) - namespace: cfx-kube-discovery [](#__codelineno-5-9)   kind: ServiceAccount [](#__codelineno-5-10)   name: cfx-api-access  [](#__codelineno-5-11) roleRef: [](#__codelineno-5-12)   apiGroup: rbac.authorization.k8s.io [](#__codelineno-5-13)   kind: ClusterRole [](#__codelineno-5-14)   name: cfx-api-access-role [](#__codelineno-5-15) [](#__codelineno-5-16) EOF`

`[](#__codelineno-6-1) kubectl apply -f cfx-api-access-role-binding.yml`

**Step-5**: Get the secret name for service user account.

`[](#__codelineno-7-1) kubectl get serviceaccount cfx-api-access -o=jsonpath='{.secrets[0].name}' -n cfx-kube-discovery`

**Step-6**: Get the secret token of service user account and decode it. It shows the HTTP bearer token for service account.

Note

The secret name of the service user account 'cfx-api-access' is provided in the command's output executed in Step-5

`[](#__codelineno-8-1) kubectl get secrets cfx-api-access-token-XXXXX -o=jsonpath='{.data.token}' -n cfx-kube-discovery | base64 -d`

**Step-7**: Verify the Kubernetes cluster's HTTP API access using the below sample curl command which should list all namespaces that are configured in the Kubernetes cluster.

`[](#__codelineno-9-1) curl -k  https://kube-cluster-ip:6443/api/v1/namespaces -H "Authorization: Bearer <bearer token>" --insecure`

****4\. Adding Kubernetes Cluster as Datasource/Extension in RDA Studio****
---------------------------------------------------------------------------

Kubernetes Cluster or any other datasource/extension's configuration is configured in RDA's user interface. Login into RDA's user interface using a browser.

**https://\[rda-ip-address\]:9998**

Info

Default username and password of standalone **RDA Studio** is **rdademo** and **rdademo1234**

Under **Notebook**, click on **CFXDX Python 3** box

![K8s_RDA_Studio](https://bot-docs.cloudfabrix.io/images/rda_integrations/kubernetes/kubernetes_kubectlpython.png)

In the '**Notebook**' command box, type botadmin() and **alt (or option) + Enter** to open datasource administration menu. Click on '**Add**' menu and under **Type** drop down, select **kubernetes-inventory**

![K8s_RDA_Datasource](https://bot-docs.cloudfabrix.io/images/rda_integrations/kubernetes/kubernetes_kubectlbotadmin.png)

Select API or SSH to access Kubernetes cluster resources for inventory data collection.

**Login Method**: API (configure the below options)

*   Hostname (IP Address or DNS name of Kubernetes cluster)
*   Port (HTTP API access port, ex: 6443)
*   Protocol (http or https)
*   API Token (Service user account's bearer token extracted in Step-6)
*   Verify SSL Certificate (Optional)

**Login Method**: SSH (configure the below options)

*   Hostname (IP Address or DNS name of Kubernetes cluster)
*   Username (Service user account for SSH access)
*   Password (Enter the user's password only when password based authentication is used)
*   Private Key Passphrase (Enter the key passphrase only when SSH key based authentication is used)
*   SSH Private Key (Enter the SSH RSA key only when SSH key based authentication is used)
*   Port (SSH port, default is set to 22)

For the details on Kubernetes cluster inventory data collection bots, refer [CloudFabrix RDA Bot Documentation](https://bot-docs.cloudfabrix.io/Extensions/extensions_F_K/#extension-kubernetes-inventory "CloudFabrix RDA Bot Documentation")

Was this page helpful?

Thanks for your feedback!

Thanks for your feedback!