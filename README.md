# k8s-jenkins-helm
## Tools

- [Jenkins](https://www.jenkins.io/)
- [Kubernetes](https://kubernetes.io/)
- [helm](https://helm.sh/)
- [K3s](https://k3s.io/)
- [MetalLB](https://metallb.universe.tf/)
- [Local Path Provisioner](https://github.com/rancher/local-path-provisioner)

## Create K8s Cluster
Please see the steps on the page.
- [Link](https://github.com/avorakh/rsschool-devops-course-tasks/blob/task-04/README.md)
- [Change, including helm installation on a bastion host](https://github.com/avorakh/rsschool-devops-course-tasks/pull/10)

## Prerequisites
1. **Clone the Repository**: Clone the project repository to your local machine using the following command:

   ```bash
   https://github.com/avorakh/k8s-jenkins-helm.git
   ```

2. **Navigate to the Project Directory**: Change a working directory to the project folder:

   ```bash
   cd k8s-jenkins-helm
   ```

## Prepare the Cluster
### 1. Installation and Configuration MetalLB on K3s (using L2)

1. **Add the MetalLB Helm repository:**

    ```bash
    helm repo add metallb https://metallb.github.io/metallb
    helm repo update
    ```

2. **Install MetalLB using Helm:**

    ```bash
    helm install metallb metallb/metallb --namespace metallb-system --create-namespace
    ```

3. **Verify MetalLB installation:**

    ```bash
    kubectl get pods -n metallb-system
    ```

    Ensure all pods are running.

4. **Create a ConfigMap for MetalLB:**

    Update file `metallb-config.yaml` the with the following content:

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    namespace: metallb-system
    name: config
    data:
    config: |
        address-pools:
        - name: default
        protocol: layer2
        addresses:
        - <PUT_PUBLIC_IP_OF_WORKER_NODE>-<PUT_PUBLIC_IP_OF_WORKER_NODE>
    ```

    This configuration uses the public IP of a worker node.

5. **Apply the ConfigMap:**

    ```bash
    kubectl apply -f ./config/metallb/metallb-config.yaml
    ```

6. **Verify MetalLB configuration:**

    ```bash
    kubectl describe configmap config -n metallb-system
    ```

    Ensure the address pool is correctly configured.

### Create a persistent volume(PV) and a persistent volume claim(PVC)

- Run the following command to apply the spec.

    ```bash
    kubectl apply -f ./config/pv-pvc/jenkins-storageclass.yaml
    ```

## Jenkins Installation and Configuration
1. **Add the Jenkins Helm repository:**

    ```bash
    helm repo add jenkinsci https://charts.jenkins.io
    helm repo update
    helm search repo jenkins
    ```

2. **Create Jenkins namespace**

    ```bash
    kubectl create namespace jenkins
    ```

3. **Download Dependencies**

    ```bash
    helm dependency update ./jenkins-ci
    ```

4. **Install Jenkins using Helm**

    ```bash
    helm install jenkins-ci  -n jenkins ./jenkins-ci
    ```

5. **Verify Jenkins installation:**

    ```bash
    kubectl get pods  -n jenkins
    kubectl logs jenkins-ci-0  -n jenkins
    kubectl get service -n jenkins
    ```

    Ensure all pods are running. Check a jenkins-ci pod log.

6. **Get the 'admin' user password**

    ```bash
    jsonpath="{.data.jenkins-admin-password}"
    secret=$(kubectl get secret -n jenkins jenkins-ci -o jsonpath=$jsonpath)
    echo $(echo $secret | base64 --decode)
    ```

7. **Visit Jenkins in the browser**

- Login with the password from step 6 and the username: admin.

    ```
    http://<PUT_PUBLIC_IP_OF_WORKER_NODE>:80
    ```

## Clean Up Jenkins

- **Run the following command to delete the Jenkins Helm deployment**
    ```bash
    helm uninstall jenkins-ci -n jenkins
    ```