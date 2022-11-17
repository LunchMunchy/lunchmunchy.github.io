---
layout: post
title: How to Setup Zero2JupyterHub like a Noob z2jh, tutorial
subtitle: DAY X
tags: [100DaysOfCloud, ]
cover-img: REPLACE_ME
thumbnail-img: assets/img/thumbnails/Day%20X.png
comments: true
readtime: true
---

| Articles                                                                                           |
| -------------------------------------------------------------------------------------------------- |
| [**Part 1: Intro + Script**](#example)                                                                 |
| [Part 2: Setting up Amazon AWS is not Intuitive](#example2)                                        |
| [Part 3: Kubenetes Terminology and Architecture is not intuitive](#third-example)                  |
| [Part 4: How to Automate your K8s Cluster Setup with Kops](#fourth-examplehttpwwwfourthexamplecom) |
| [Part 5: KubeCTL and Helm](https://lunchmunchy.github.io/aboutme/)                                 |
| [Part 6: What I’m going to do next](https://lunchmunchy.github.io/aboutme/)                        |
   
---
> BLUF:
> 
> **If you’re here for the script, [scroll down](#-the-script-z2jhsh).**
> 
> If you care about me for the quality content I write, you can read through my experience setting up JupyterHub on K8s. 
> 
> This is a multi-blog series. That’s right, I’m milking this sucker for as much content as I can get.

# 📚 Table of Contents (Part 1)
- [📚 Table of Contents (Part 1)](#-table-of-contents-part-1)
- [✌️ Introduction](#️-introduction)
- [🤓 The Script: z2jh.sh](#-the-script-z2jhsh)
- [🕺 Breakdown](#-breakdown)
  - [How to use the script](#how-to-use-the-script)
  - [Script Explanation](#script-explanation)
  - [Points of Failure](#points-of-failure)
  - [Environment](#environment)
- [🙏 Hope this helped](#-hope-this-helped)

# ✌️ Introduction
>*My name is Paul and I’m a very junior Cloud engineering. I decided to document my learning for fun [and profit](https://lunchmunchy.github.io/aboutme/). My writing style is beginner friendly, I hope. If you’re totally new, then maybe you'll learn from my mistakes.*

It’s been a while since I’ve blogged. I look at my first tweet ever and pat myself on the back. I’m a freaking psychic.

<div class="center">
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">First tweet ever is <a href="https://twitter.com/hashtag/100DaysOfKubernetes?src=hash&amp;ref_src=twsrc%5Etfw">#100DaysOfKubernetes</a>! 🥳<br><br>One of two things will happen:<br>1. I somehow complete this with relative consistency<br>2. You see my tweets disappear after &lt; 2 weeks<br><br>I&#39;m putting money on #2. Yay...<br><br>I&#39;ll look back on this 10 years later and cringe 🤮<a href="https://twitter.com/hashtag/DevOps?src=hash&amp;ref_src=twsrc%5Etfw">#DevOps</a></p>&mdash; Paul Tan (@LunchMunchy) <a href="https://twitter.com/LunchMunchy/status/1567162164065992704?ref_src=twsrc%5Etfw">September 6, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
</div>

But much to my surprise, I didn’t *actually* quit my cloud learning.

Last week to this day, I took on a DevSecOps project. My goal was simple: Set up JupyterHub on Kubernetes. I took nearly a week to follow the (un)official (Z2JH Guide)[https://z2jh.jupyter.org/en/stable/], and let me tell you: It was a mess. That’s why I’m blogging about everything that went wrong. On the bright side, I learned a whole bunch, and I feel like 1% more like a true cloud engineer now.

Also, I turned what I learned into a simple bash script that should get you a very basic working JupyterHub. In my opinion, the (Z2JH Guide)[https://z2jh.jupyter.org/en/stable/] really wasn’t that intuitive or great, especially for beginners. That’s why I hope folks will get a lot of use from (Part 3)[] and (Part 4)[] of my writeup.


# 🤓 The Script: z2jh.sh

``` bash
#!/bin/bash

install_tools() {
sudo apt install curl unzip

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x ./kops
sudo mv ./kops /usr/local/bin/

curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# aws --version
# kops --version
# kubectl --version
# helm version
}

setup_aws_cli() {
  echo -e "Make sure to get the following documents:\n
  Account ID
  Username
  Password
  Access Key
  Secret Access Key"

  export PATH=/usr/local/bin/:$PATH
  source ~/.bash_profile || source ~/.bashrc|| source ~/.profile
  complete -C '/usr/local/bin/aws_completer' aws

  aws configure
  
  echo State S3 bucket name:
  read bucketname
  aws s3api create-bucket --bucket "$bucketname" --region us-east-1
}

create_k8s_cluster() {
ssh-keygen
kops create secret --name $NAME sshpublickey admin -i ~/.ssh/id_rsa.pub

echo Assign a cluster name:
read clustername
export NAME="$clustername".k8s.local

export KOPS_STATE_STORE=s3://"$bucketname"

export REGION=`curl -s http://169.254.169.254/latest/dynamic/instance-identity/document|grep region|awk -F\" '{print $4}'`

export ZONES=$(aws ec2 describe-availability-zones --region $REGION | grep ZoneName | awk '{print $2}' | head -n1 | tr -d '"' | sed 's/.$//')

kops create cluster \
--name "$NAME" \
--zones "$ZONES" \
--state "$KOPS_STATE_STORE" \
--yes

kops validate cluster --wait 15m

kubectl config set-context $(kubectl config current-context) --namespace $NAME

cat << EOF > storageclass.yml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  annotations:
    storageclass.beta.kubernetes.io/is-default-class: "true"
  name: gp2
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  encrypted: "true"
EOF
kubectl apply -f storageclass.yml
}

install_jupyterhub() {
cat <<EOF >config.yaml
# This file can update the JupyterHub Helm chart's default configuration values.
#
# For reference see the configuration reference and default values, but make
# sure to refer to the Helm chart version of interest to you!
#
# Introduction to YAML:     https://www.youtube.com/watch?v=cdLNKUoMc6c
e Chart config reference:   https://zero-to-jupyterhub.readthedocs.io/en/stable/resources/reference.html
# Chart default values:     https://github.com/jupyterhub/zero-to-jupyterhub-k8s/blob/HEAD/jupyterhub/values.yaml
# Available chart versions: https://jupyterhub.github.io/helm-chart/
#
EOF

helm repo add jupyterhub https://jupyterhub.github.io/helm-chart/
helm repo update
helm search repo jupyterhub

# kubectl get storageclass
# kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"false"}}}'

echo State release name:
read releasename

echo State namespace:
read namespace
export NAMESPACE="$namespace"

helm upgrade --cleanup-on-fail \
  --install "$releasename" jupyterhub/jupyterhub \
  --namespace $NAMESPACE \
  --create-namespace \
  --version=2.0.0 \
  --values config.yaml

sleep 30
kubectl get pod --namespace $NAMESPACE
kubectl --namespace $NAMESPACE get service proxy-public #--output jsonpath='{.status.loadBalancer.ingress[].ip}'
}

setup_aws
install_kops_kubectl
setup_kops_cluster
install_jupyterhub


echo "All Done!"
```

# 🕺 Breakdown
## How to use the script
**Step 0.** 

I'll assumes you have your Amazon AWS account all set up. If not, read [Part 2](#example2) of my blog
You will need the following creds ready to copy + paste:

- Access Key
- Secret Acccess Key

**Step 1.**
```
chmod +x z2jh.sh
./z2jh.sh
```

**Step 2.**
- Type in your root password when prompted
- Hit enter multiple times to create your ssh-key
- Type in your bucketname, clustername, releasename, and namespace when prompted (Explained in [Part 4](https://lunchmunchy.github.io/aboutme/) and [Part 5](https://lunchmunchy.github.io/aboutme/))

**Step 3.**

Copy the proxy-public IP address from the final output, and navigate to the website on your browser.

**Step 4.**
``` python
if error:
  troubleshoot()
else:
  rejoice()
```

## Script Explanation
This is a bash script consisting of 4 functions:

- `install_tools` - Installs the following packages: `curl`, `unzip`, `aws`, `kops`, `kubectl`, `helm`
- `setup_aws_cli` - Gets your linux environment compatible with aws cli. This is a requisite for `kops` to work. 
- `create_k8s_cluster` - Creates an AWS EC2 Kubernetes cluster (1 master + 1 node) with dynamic storage (for creating new pods everytime a new JupyterHub user log in)
- `install_jupyterhub` - Uses `helm` to install JupyterHub on your brand new cluster.

## Points of Failure
Here are some points where you may need to troubleshoot, or do things manually.

1. Line X: You used a bucket name that's not unique, meaning one of the millions of Amazon AWS users already claimed the name.
2. Line X: 
3. Line X: 
4. Line X: 
5. Line X: 
6. 

If things still don't work, I will just assume it's all your fault because you've got a messed up...

## Environment
This is what worked for me, you don't have to replicate me exactly. I think as long as you use `Ubuntu 20.04` and up along side `bash`, you'll be fine.

| Thing             | Type                         |
| ----------------- | ---------------------------- |
| Date              | 2022-11-17                   |
| Virtual Machine   | Virt-Manager                 |
| Linux             | Lubuntu 20.04.1 x86_64       |
| Shell             | `bash` 5.1.16                |
| `aws` Version     | aws-cli/2.8.12 Python/3.9.11 |
| `kubectl` Version | Bash                         |
| `Kops` Version    | 1.25                         |
| `Helm` Version    | Bash                         |


# 🙏 Hope this helped
Or keep reading the next pages of my blog to troubleshoot.

| Articles                                                                                           |
| -------------------------------------------------------------------------------------------------- |
| [Part 1: Intro + Script](#example)                                                                 |
| [Part 2: Setting up Amazon AWS is not Intuitive](#example2)                                        |
| [Part 3: Kubenetes Terminology and Architecture is not intuitive](#third-example)                  |
| [Part 4: How to Automate your K8s Cluster Setup with Kops](#fourth-examplehttpwwwfourthexamplecom) |
| [Part 5: KubeCTL and Helm](https://lunchmunchy.github.io/aboutme/)                                 |
| [Part 6: What I’m going to do next](https://lunchmunchy.github.io/aboutme/)                        |