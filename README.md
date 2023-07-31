# Kubernetes-Zero-to-Hero
Creating this repo with an intent to make Kubernetes easy for begineers. This is a work-in-progress repo.

### K8s-Arch

![image](https://github.com/Mallik-Raj/Kubernetes-Zero-to-Hero/assets/53124649/b0f6f558-76ee-474f-b71c-33e9c7ff834d)




## Kubernetes Installation Using KOPS on EC2

### Create an EC2 instance or use your personal laptop.

Dependencies required 

1. Python3
2. AWS CLI
3. kubectl

###  Install dependencies

```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

```
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
```

```
sudo apt-get update
sudo apt-get install -y python3-pip apt-transport-https kubectl
```

```
pip3 install awscli --upgrade
```

```
export PATH="$PATH:/home/ubuntu/.local/bin/"
```

### Install KOPS (our hero for today)

```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64

chmod +x kops-linux-amd64

sudo mv kops-linux-amd64 /usr/local/bin/kops
```

### Provide the below permissions to your IAM user. If you are using the admin user, the below permissions are available by default

1. AmazonEC2FullAccess
2. AmazonS3FullAccess
3. IAMFullAccess
4. AmazonVPCFullAccess

### Set up AWS CLI configuration on your EC2 Instance or Laptop.

Run `aws configure`

## Kubernetes Cluster Installation 

Please follow the steps carefully and read each command before executing.

### Create S3 bucket for storing the KOPS objects.

```
aws s3api create-bucket --bucket kops-abhi-storage --region us-east-1
```

### Create the cluster 

```
kops create cluster --name=demok8scluster.k8s.local --state=s3://kops-abhi-storage --zones=us-east-1a --node-count=1 --node-size=t2.micro --master-size=t2.micro  --master-volume-size=8 --node-volume-size=8
```

### Important: Edit the configuration as there are multiple resources created which won't fall into the free tier.

```
kops edit cluster myfirstcluster.k8s.local
```

Step 12: Build the cluster

```
kops update cluster demok8scluster.k8s.local --yes --state=s3://kops-abhi-storage
```

This will take a few minutes to create............

After a few mins, run the below command to verify the cluster installation.

```
kops validate cluster demok8scluster.k8s.local
```

## Difference between Container and POD

Container is a lightweight, standalone executable package that contains everything needed to run an application, including code, libraries, and dependencies. Containers provide a consistent, isolated environment for your applications, making it easier to move them between different environments without worrying about compatibility issues.

A pod is the smallest deployable unit in Kubernetes, which is a popular container orchestration platform. A pod is a logical host for one or more containers and provides a shared network namespace and storage volumes for those containers. In other words, a pod is a way to group together one or more containers that need to work together as a single unit. Each pod has a unique IP address and can communicate with other pods and services in the Kubernetes cluster.

![pod-animation-kubernetes](https://github.com/Mallik-Raj/Kubernetes-Zero-to-Hero/assets/53124649/30d4f299-69c8-4d5b-8361-0e9dd4d39624)



### Commands for POD

```
kubectl apply -f simple-pod.yaml              # create a pod
kubectl get pods --all-namespaces             # List all pods in all namespaces
kubectl get pods -o wide                      # List all pods in the current namespace, with more details
kubectl get pods                              # List all pods in the namespace
kubectl get pod my-pod -o yaml                # Get a pod's YAML
```

#### Example of a Pod which consists of a container running the image
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

## What is a Deployment in K8S?
A deployment is an object in Kubernetes that lets you manage a set of identical pods.Without a deployment, you’d need to create, update, and delete a bunch of pods manually.With a deployment, you declare a single object in a YAML file. This object is responsible for creating the pods, making sure they stay up to date, and ensuring there are enough of them running ,You can also easily autoscale your applications using a Kubernetes deployment.Using this we will be able to achivce autohealing.


![deployment-diagram-kubernetes](https://github.com/Mallik-Raj/Kubernetes-Zero-to-Hero/assets/53124649/caa8abe8-e263-4c44-8b04-772c541e38ca)


### YAML reference example

![deployment-yaml-diagram](https://github.com/Mallik-Raj/Kubernetes-Zero-to-Hero/assets/53124649/9c9bed37-0df4-4085-a4c9-e98bf460335c)


First, the `replicas` key sets the number of instances of the pod that the deployment should keep alive.
Next, we use a label selector to tell the deployment which pods are part of the deployment. This essentially says "all the pods matching these labels are grouped in our deployment."
After that, we have the `template` object.
This is interesting. It’s essentially a pod template jammed inside our deployment spec. When the deployment creates pods, it will create them using this template!

So everything under the template key is a regular pod specification.
In this case, the deployment will create pods that run  `nginx-hostname` and with the configured labels.

##### How to create a deployment in Kubernetes
If you want create your deployment from a file, you can use `kubectl apply -f deployment.yaml` to create your deployment.

If want create your deployment from the command line, you can use `kubectl apply deployment my-deployment` to create your deployment
##### How to delete a deployment in Kubernetes
If you’ve created your deployment from a file, you can use `kubectl delete -f deployment.yaml` to delete your deployment.

If you’ve created your deployment from the command line, you can use `kubectl delete deployment my-deployment` to delete your deployment.








