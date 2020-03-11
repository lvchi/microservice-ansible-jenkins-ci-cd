# Setup Kubernetes (K8s) Cluster on AWS


1. Create Ubuntu EC2 instance
1. install AWSCLI
   ```sh
    curl https://s3.amazonaws.com/aws-cli/awscli-bundle.zip -o awscli-bundle.zip
    apt install unzip python
    unzip awscli-bundle.zip
    #sudo apt-get install unzip - if you dont have unzip in your system
    ./awscli-bundle/install -i /usr/local/aws -b /usr/local/bin/aws
    ```

1. Install kubectl on ubuntu instance
   ```sh
   curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
   ```

1. Install kops on ubuntu instance
   ```sh
    curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x kops-linux-amd64
    sudo mv kops-linux-amd64 /usr/local/bin/kops
    ```
1. Create an IAM user/role  with Route53, EC2, IAM and S3 full access

1. Attach IAM role to ubuntu instance
   ```sh
   # Note: If you create IAM user with programmatic access then provide Access keys. Otherwise region information is enough
   aws configure
    ```

1. Create a Route53 private hosted zone (you can create Public hosted zone if you have a domain)
   ```sh
   Routeh53 --> hosted zones --> created hosted zone  
   Domain Name: valaxy.net
   Type: Private hosted zone for Amzon VPC
   ```

1. create an S3 bucket
   ```sh
    aws s3 mb s3://demo.valaxy.net
   ```
1. Expose environment variable:
   ```sh
    export KOPS_STATE_STORE=s3://demo.valaxy.net
   ```

1. Create sshkeys before creating cluster
   ```sh
    ssh-keygen
   ```

1. Create kubernetes cluster definitions on S3 bucket
   ```sh
   kops create cluster --cloud=aws --zones=us-east-1b --name=demo.valaxy.net --dns-zone=valaxy.net --dns private 
    ```

1. Create kubernetes cluser
    ```sh
    kops update cluster demo.k8s.valaxy.net --yes
    ```

1. Validate your cluster
     ```sh
      kops validate cluster
    ```

1. To list nodes
   ```sh
   kubectl get nodes
   ```

1. To delete cluster
    ```sh
     kops delete cluster demo.k8s.valaxy.net --yes
    ```
   
#### Deploying Nginx pods on Kubernetes
1. Deploying Nginx Container
    ```sh
    kubectl run sample-nginx --image=nginx --replicas=2 --port=80
    # kubectl run simple-devops-project --image=yankils/simple-devops-image --replicas=2 --port=8080
    kubectl get pods
    kubectl get deployments
   ```

1. Expose the deployment as service. This will create an ELB in front of those 2 containers and allow us to publicly access them.
   ```sh
   kubectl expose deployment sample-nginx --port=80 --type=LoadBalancer
   # kubectl expose deployment simple-devops-project --port=8080 --type=LoadBalancer
   kubectl get services -o wide
   ```

----------------------log----------------------------------------
root@ip-10-0-2-78:/home/ubuntu# export KOPS_STATE_STORE=s3://vote.valaxy.net
root@ip-10-0-2-78:/home/ubuntu# kops create cluster --cloud=aws --zones=us-east-1b --name=vote.valaxy.net --dns-zone=valaxy.net --dns private
I0311 07:16:12.377415    1546 subnets.go:184] Assigned CIDR 172.20.32.0/19 to subnet us-east-1b
I0311 07:16:13.447460    1546 create_cluster.go:1568] Using SSH public key: /root/.ssh/id_rsa.pub
Previewing changes that will be made:

I0311 07:16:14.104594    1546 dns.go:94] Private DNS: skipping DNS validation
I0311 07:16:14.228498    1546 executor.go:103] Tasks: 0 done / 87 total; 44 can run
I0311 07:16:15.250878    1546 executor.go:103] Tasks: 44 done / 87 total; 23 can run
I0311 07:16:15.543157    1546 executor.go:103] Tasks: 67 done / 87 total; 18 can run
I0311 07:16:15.649243    1546 executor.go:103] Tasks: 85 done / 87 total; 2 can run
I0311 07:16:15.722530    1546 executor.go:103] Tasks: 87 done / 87 total; 0 can run
Will create resources:
  AutoscalingGroup/master-us-east-1b.masters.vote.valaxy.net
        Granularity             1Minute
        LaunchConfiguration     name:master-us-east-1b.masters.vote.valaxy.net
        MaxSize                 1
        Metrics                 [GroupDesiredCapacity, GroupInServiceInstances, GroupMaxSize, GroupMinSize, GroupPendingInstances, GroupStandbyInstances, GroupTerminatingInstances, GroupTotalInstances]
        MinSize                 1
        Subnets                 [name:us-east-1b.vote.valaxy.net]
        SuspendProcesses        []
        Tags                    {k8s.io/cluster-autoscaler/node-template/label/kops.k8s.io/instancegroup: master-us-east-1b, k8s.io/role/master: 1, kops.k8s.io/instancegroup: master-us-east-1b, Name: master-us-east-1b.masters.vote.valaxy.net, KubernetesCluster: vote.valaxy.net}

  AutoscalingGroup/nodes.vote.valaxy.net
        Granularity             1Minute
        LaunchConfiguration     name:nodes.vote.valaxy.net
        MaxSize                 2
        Metrics                 [GroupDesiredCapacity, GroupInServiceInstances, GroupMaxSize, GroupMinSize, GroupPendingInstances, GroupStandbyInstances, GroupTerminatingInstances, GroupTotalInstances]
        MinSize                 2
        Subnets                 [name:us-east-1b.vote.valaxy.net]
        SuspendProcesses        []
        Tags                    {k8s.io/cluster-autoscaler/node-template/label/kops.k8s.io/instancegroup: nodes, k8s.io/role/node: 1, kops.k8s.io/instancegroup: nodes, Name: nodes.vote.valaxy.net, KubernetesCluster: vote.valaxy.net}

  DHCPOptions/vote.valaxy.net
        DomainName              ec2.internal
        DomainNameServers       AmazonProvidedDNS
        Shared                  false
        Tags                    {kubernetes.io/cluster/vote.valaxy.net: owned, Name: vote.valaxy.net, KubernetesCluster: vote.valaxy.net}

  EBSVolume/b.etcd-events.vote.valaxy.net
        AvailabilityZone        us-east-1b
        Encrypted               false
        SizeGB                  20
        Tags                    {k8s.io/etcd/events: b/b, k8s.io/role/master: 1, kubernetes.io/cluster/vote.valaxy.net: owned, Name: b.etcd-events.vote.valaxy.net, KubernetesCluster: vote.valaxy.net}
        VolumeType              gp2

  EBSVolume/b.etcd-main.vote.valaxy.net
        AvailabilityZone        us-east-1b
        Encrypted               false
        SizeGB                  20
        Tags                    {kubernetes.io/cluster/vote.valaxy.net: owned, Name: b.etcd-main.vote.valaxy.net, KubernetesCluster: vote.valaxy.net, k8s.io/etcd/main: b/b, k8s.io/role/master: 1}
        VolumeType              gp2

  IAMInstanceProfile/masters.vote.valaxy.net
        Shared                  false

  IAMInstanceProfile/nodes.vote.valaxy.net
        Shared                  false

  IAMInstanceProfileRole/masters.vote.valaxy.net
        InstanceProfile         name:masters.vote.valaxy.net id:masters.vote.valaxy.net
        Role                    name:masters.vote.valaxy.net

  IAMInstanceProfileRole/nodes.vote.valaxy.net
        InstanceProfile         name:nodes.vote.valaxy.net id:nodes.vote.valaxy.net
        Role                    name:nodes.vote.valaxy.net

  IAMRole/masters.vote.valaxy.net
        ExportWithID            masters

  IAMRole/nodes.vote.valaxy.net
        ExportWithID            nodes

  IAMRolePolicy/masters.vote.valaxy.net
        Role                    name:masters.vote.valaxy.net

  IAMRolePolicy/nodes.vote.valaxy.net
        Role                    name:nodes.vote.valaxy.net

  InternetGateway/vote.valaxy.net
        VPC                     name:vote.valaxy.net
        Shared                  false
        Tags                    {kubernetes.io/cluster/vote.valaxy.net: owned, Name: vote.valaxy.net, KubernetesCluster: vote.valaxy.net}

  Keypair/apiserver-aggregator
        Signer                  name:apiserver-aggregator-ca id:cn=apiserver-aggregator-ca
        Subject                 cn=aggregator
        Type                    client
        Format                  v1alpha2

  Keypair/apiserver-aggregator-ca
        Subject                 cn=apiserver-aggregator-ca
        Type                    ca
        Format                  v1alpha2

  Keypair/apiserver-proxy-client
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=apiserver-proxy-client
        Type                    client
        Format                  v1alpha2

  Keypair/ca
        Subject                 cn=kubernetes
        Type                    ca
        Format                  v1alpha2

  Keypair/etcd-clients-ca
        Subject                 cn=etcd-clients-ca
        Type                    ca
        Format                  v1alpha2

  Keypair/etcd-manager-ca-events
        Subject                 cn=etcd-manager-ca-events
        Type                    ca
        Format                  v1alpha2

  Keypair/etcd-manager-ca-main
        Subject                 cn=etcd-manager-ca-main
        Type                    ca
        Format                  v1alpha2

  Keypair/etcd-peers-ca-events
        Subject                 cn=etcd-peers-ca-events
        Type                    ca
        Format                  v1alpha2

  Keypair/etcd-peers-ca-main
        Subject                 cn=etcd-peers-ca-main
        Type                    ca
        Format                  v1alpha2

  Keypair/kops
        Signer                  name:ca id:cn=kubernetes
        Subject                 o=system:masters,cn=kops
        Type                    client
        Format                  v1alpha2

  Keypair/kube-controller-manager
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=system:kube-controller-manager
        Type                    client
        Format                  v1alpha2

  Keypair/kube-proxy
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=system:kube-proxy
        Type                    client
        Format                  v1alpha2

  Keypair/kube-scheduler
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=system:kube-scheduler
        Type                    client
        Format                  v1alpha2

  Keypair/kubecfg
        Signer                  name:ca id:cn=kubernetes
        Subject                 o=system:masters,cn=kubecfg
        Type                    client
        Format                  v1alpha2

  Keypair/kubelet
        Signer                  name:ca id:cn=kubernetes
        Subject                 o=system:nodes,cn=kubelet
        Type                    client
        Format                  v1alpha2

  Keypair/kubelet-api
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=kubelet-api
        Type                    client
        Format                  v1alpha2

  Keypair/master
        AlternateNames          [100.64.0.1, 127.0.0.1, api.internal.vote.valaxy.net, api.vote.valaxy.net, kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local]
        Signer                  name:ca id:cn=kubernetes
        Subject                 cn=kubernetes-master
        Type                    server
        Format                  v1alpha2

  LaunchConfiguration/master-us-east-1b.masters.vote.valaxy.net
        AssociatePublicIP       true
        IAMInstanceProfile      name:masters.vote.valaxy.net id:masters.vote.valaxy.net
        ImageID                 kope.io/k8s-1.16-debian-stretch-amd64-hvm-ebs-2020-01-17
        InstanceType            m3.medium
        RootVolumeDeleteOnTermination   true
        RootVolumeSize          64
        RootVolumeType          gp2
        SSHKey                  name:kubernetes.vote.valaxy.net-d8:5d:be:e5:e3:77:cf:bd:ec:a8:9d:74:4b:ec:51:f7 id:kubernetes.vote.valaxy.net-d8:5d:be:e5:e3:77:cf:bd:ec:a8:9d:74:4b:ec:51:f7
        SecurityGroups          [name:masters.vote.valaxy.net]
        SpotPrice

  LaunchConfiguration/nodes.vote.valaxy.net
        AssociatePublicIP       true
        IAMInstanceProfile      name:nodes.vote.valaxy.net id:nodes.vote.valaxy.net
        ImageID                 kope.io/k8s-1.16-debian-stretch-amd64-hvm-ebs-2020-01-17
        InstanceType            t2.medium
        RootVolumeDeleteOnTermination   true
        RootVolumeSize          128
        RootVolumeType          gp2
        SSHKey                  name:kubernetes.vote.valaxy.net-d8:5d:be:e5:e3:77:cf:bd:ec:a8:9d:74:4b:ec:51:f7 id:kubernetes.vote.valaxy.net-d8:5d:be:e5:e3:77:cf:bd:ec:a8:9d:74:4b:ec:51:f7
        SecurityGroups          [name:nodes.vote.valaxy.net]
        SpotPrice

  ManagedFile/etcd-cluster-spec-events
        Location                backups/etcd/events/control/etcd-cluster-spec

  ManagedFile/etcd-cluster-spec-main
        Location                backups/etcd/main/control/etcd-cluster-spec

  ManagedFile/manifests-etcdmanager-events
        Location                manifests/etcd/events.yaml

  ManagedFile/manifests-etcdmanager-main
        Location                manifests/etcd/main.yaml

  ManagedFile/vote.valaxy.net-addons-bootstrap
        Location                addons/bootstrap-channel.yaml

  ManagedFile/vote.valaxy.net-addons-core.addons.k8s.io
        Location                addons/core.addons.k8s.io/v1.4.0.yaml

  ManagedFile/vote.valaxy.net-addons-dns-controller.addons.k8s.io-k8s-1.12
        Location                addons/dns-controller.addons.k8s.io/k8s-1.12.yaml

  ManagedFile/vote.valaxy.net-addons-dns-controller.addons.k8s.io-k8s-1.6
        Location                addons/dns-controller.addons.k8s.io/k8s-1.6.yaml

  ManagedFile/vote.valaxy.net-addons-dns-controller.addons.k8s.io-pre-k8s-1.6
        Location                addons/dns-controller.addons.k8s.io/pre-k8s-1.6.yaml

  ManagedFile/vote.valaxy.net-addons-kops-controller.addons.k8s.io-k8s-1.16
        Location                addons/kops-controller.addons.k8s.io/k8s-1.16.yaml

  ManagedFile/vote.valaxy.net-addons-kube-dns.addons.k8s.io-k8s-1.12
        Location                addons/kube-dns.addons.k8s.io/k8s-1.12.yaml

  ManagedFile/vote.valaxy.net-addons-kube-dns.addons.k8s.io-k8s-1.6
        Location                addons/kube-dns.addons.k8s.io/k8s-1.6.yaml

  ManagedFile/vote.valaxy.net-addons-kube-dns.addons.k8s.io-pre-k8s-1.6
        Location                addons/kube-dns.addons.k8s.io/pre-k8s-1.6.yaml

  ManagedFile/vote.valaxy.net-addons-kubelet-api.rbac.addons.k8s.io-k8s-1.9
        Location                addons/kubelet-api.rbac.addons.k8s.io/k8s-1.9.yaml

  ManagedFile/vote.valaxy.net-addons-limit-range.addons.k8s.io
        Location                addons/limit-range.addons.k8s.io/v1.5.0.yaml

  ManagedFile/vote.valaxy.net-addons-rbac.addons.k8s.io-k8s-1.8
        Location                addons/rbac.addons.k8s.io/k8s-1.8.yaml

  ManagedFile/vote.valaxy.net-addons-storage-aws.addons.k8s.io-v1.15.0
        Location                addons/storage-aws.addons.k8s.io/v1.15.0.yaml

  ManagedFile/vote.valaxy.net-addons-storage-aws.addons.k8s.io-v1.6.0
        Location                addons/storage-aws.addons.k8s.io/v1.6.0.yaml

  ManagedFile/vote.valaxy.net-addons-storage-aws.addons.k8s.io-v1.7.0
        Location                addons/storage-aws.addons.k8s.io/v1.7.0.yaml

  Route/0.0.0.0/0
        RouteTable              name:vote.valaxy.net
        CIDR                    0.0.0.0/0
        InternetGateway         name:vote.valaxy.net

  RouteTable/vote.valaxy.net
        VPC                     name:vote.valaxy.net
        Shared                  false
        Tags                    {KubernetesCluster: vote.valaxy.net, kubernetes.io/cluster/vote.valaxy.net: owned, kubernetes.io/kops/role: public, Name: vote.valaxy.net}

  RouteTableAssociation/us-east-1b.vote.valaxy.net
        RouteTable              name:vote.valaxy.net
        Subnet                  name:us-east-1b.vote.valaxy.net

  SSHKey/kubernetes.vote.valaxy.net-d8:5d:be:e5:e3:77:cf:bd:ec:a8:9d:74:4b:ec:51:f7
        KeyFingerprint          33:df:a8:06:a9:ee:32:eb:18:77:ab:69:d4:88:cc:0a

  Secret/admin

  Secret/kube

  Secret/kube-proxy

  Secret/kubelet

  Secret/system:controller_manager

  Secret/system:dns

  Secret/system:logging

  Secret/system:monitoring

  Secret/system:scheduler

  SecurityGroup/masters.vote.valaxy.net
        Description             Security group for masters
        VPC                     name:vote.valaxy.net
        RemoveExtraRules        [port=22, port=443, port=2380, port=2381, port=4001, port=4002, port=4789, port=179]
        Tags                    {Name: masters.vote.valaxy.net, KubernetesCluster: vote.valaxy.net, kubernetes.io/cluster/vote.valaxy.net: owned}

  SecurityGroup/nodes.vote.valaxy.net
        Description             Security group for nodes
        VPC                     name:vote.valaxy.net
        RemoveExtraRules        [port=22]
        Tags                    {Name: nodes.vote.valaxy.net, KubernetesCluster: vote.valaxy.net, kubernetes.io/cluster/vote.valaxy.net: owned}

  SecurityGroupRule/all-master-to-master
        SecurityGroup           name:masters.vote.valaxy.net
        SourceGroup             name:masters.vote.valaxy.net

  SecurityGroupRule/all-master-to-node
        SecurityGroup           name:nodes.vote.valaxy.net
        SourceGroup             name:masters.vote.valaxy.net

  SecurityGroupRule/all-node-to-node
        SecurityGroup           name:nodes.vote.valaxy.net
        SourceGroup             name:nodes.vote.valaxy.net

  SecurityGroupRule/https-external-to-master-0.0.0.0/0
        SecurityGroup           name:masters.vote.valaxy.net
        CIDR                    0.0.0.0/0
        Protocol                tcp
        FromPort                443
        ToPort                  443

  SecurityGroupRule/master-egress
        SecurityGroup           name:masters.vote.valaxy.net
        CIDR                    0.0.0.0/0
        Egress                  true

  SecurityGroupRule/node-egress
        SecurityGroup           name:nodes.vote.valaxy.net
        CIDR                    0.0.0.0/0
        Egress                  true

  SecurityGroupRule/node-to-master-tcp-1-2379
        SecurityGroup           name:masters.vote.valaxy.net
        Protocol                tcp
        FromPort                1
        ToPort                  2379
        SourceGroup             name:nodes.vote.valaxy.net

  SecurityGroupRule/node-to-master-tcp-2382-4000
        SecurityGroup           name:masters.vote.valaxy.net
        Protocol                tcp
        FromPort                2382
        ToPort                  4000
        SourceGroup             name:nodes.vote.valaxy.net

  SecurityGroupRule/node-to-master-tcp-4003-65535
        SecurityGroup           name:masters.vote.valaxy.net
        Protocol                tcp
        FromPort                4003
        ToPort                  65535
        SourceGroup             name:nodes.vote.valaxy.net

  SecurityGroupRule/node-to-master-udp-1-65535
        SecurityGroup           name:masters.vote.valaxy.net
        Protocol                udp
        FromPort                1
        ToPort                  65535
        SourceGroup             name:nodes.vote.valaxy.net

  SecurityGroupRule/ssh-external-to-master-0.0.0.0/0
        SecurityGroup           name:masters.vote.valaxy.net
        CIDR                    0.0.0.0/0
        Protocol                tcp
        FromPort                22
        ToPort                  22

  SecurityGroupRule/ssh-external-to-node-0.0.0.0/0
        SecurityGroup           name:nodes.vote.valaxy.net
        CIDR                    0.0.0.0/0
        Protocol                tcp
        FromPort                22
        ToPort                  22

  Subnet/us-east-1b.vote.valaxy.net
        ShortName               us-east-1b
        VPC                     name:vote.valaxy.net
        AvailabilityZone        us-east-1b
        CIDR                    172.20.32.0/19
        Shared                  false
        Tags                    {Name: us-east-1b.vote.valaxy.net, KubernetesCluster: vote.valaxy.net, kubernetes.io/cluster/vote.valaxy.net: owned, SubnetType: Public, kubernetes.io/role/elb: 1}

  VPC/vote.valaxy.net
        CIDR                    172.20.0.0/16
        EnableDNSHostnames      true
        EnableDNSSupport        true
        Shared                  false
        Tags                    {kubernetes.io/cluster/vote.valaxy.net: owned, Name: vote.valaxy.net, KubernetesCluster: vote.valaxy.net}

  VPCDHCPOptionsAssociation/vote.valaxy.net
        VPC                     name:vote.valaxy.net
        DHCPOptions             name:vote.valaxy.net

Will modify resources:
  DNSZone/valaxy.net
        PrivateVPC               <nil> -> name:vote.valaxy.net

Must specify --yes to apply changes

Cluster configuration has been created.

Suggestions:
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster vote.valaxy.net
 * edit your node instance group: kops edit ig --name=vote.valaxy.net nodes
 * edit your master instance group: kops edit ig --name=vote.valaxy.net master-us-east-1b

Finally configure your cluster with: kops update cluster --name vote.valaxy.net --yes

root@ip-10-0-2-78:/home/ubuntu# kops edit cluster vote.valaxy.net
Edit cancelled, no changes made.
root@ip-10-0-2-78:/home/ubuntu# kops update cluster --name vote.valaxy.net --yes
I0311 07:17:53.520778    1571 dns.go:94] Private DNS: skipping DNS validation
I0311 07:17:53.849971    1571 executor.go:103] Tasks: 0 done / 87 total; 44 can run
I0311 07:17:55.018686    1571 vfs_castore.go:729] Issuing new certificate: "etcd-clients-ca"
I0311 07:17:55.794729    1571 vfs_castore.go:729] Issuing new certificate: "etcd-manager-ca-events"
I0311 07:17:56.291695    1571 vfs_castore.go:729] Issuing new certificate: "etcd-manager-ca-main"
I0311 07:17:56.694156    1571 vfs_castore.go:729] Issuing new certificate: "ca"
I0311 07:17:56.800970    1571 vfs_castore.go:729] Issuing new certificate: "apiserver-aggregator-ca"
I0311 07:17:56.817063    1571 vfs_castore.go:729] Issuing new certificate: "etcd-peers-ca-main"
I0311 07:17:56.974875    1571 vfs_castore.go:729] Issuing new certificate: "etcd-peers-ca-events"
I0311 07:17:57.280471    1571 executor.go:103] Tasks: 44 done / 87 total; 23 can run
I0311 07:17:58.737692    1571 vfs_castore.go:729] Issuing new certificate: "kube-proxy"
I0311 07:17:59.175969    1571 vfs_castore.go:729] Issuing new certificate: "kube-scheduler"
I0311 07:17:59.254976    1571 vfs_castore.go:729] Issuing new certificate: "kubelet-api"
I0311 07:17:59.602670    1571 vfs_castore.go:729] Issuing new certificate: "master"
I0311 07:17:59.681112    1571 vfs_castore.go:729] Issuing new certificate: "kops"
I0311 07:17:59.918997    1571 vfs_castore.go:729] Issuing new certificate: "kubelet"
I0311 07:18:00.134896    1571 vfs_castore.go:729] Issuing new certificate: "kube-controller-manager"
I0311 07:18:00.473186    1571 vfs_castore.go:729] Issuing new certificate: "apiserver-aggregator"
I0311 07:18:00.577361    1571 vfs_castore.go:729] Issuing new certificate: "apiserver-proxy-client"
I0311 07:18:00.611261    1571 vfs_castore.go:729] Issuing new certificate: "kubecfg"
I0311 07:18:01.103626    1571 executor.go:103] Tasks: 67 done / 87 total; 18 can run
I0311 07:18:01.337893    1571 launchconfiguration.go:375] waiting for IAM instance profile "nodes.vote.valaxy.net" to be ready
I0311 07:18:11.663807    1571 executor.go:103] Tasks: 85 done / 87 total; 2 can run
I0311 07:18:12.224364    1571 executor.go:103] Tasks: 87 done / 87 total; 0 can run
I0311 07:18:12.224586    1571 dns.go:155] Pre-creating DNS records
I0311 07:18:13.271398    1571 update_cluster.go:305] Exporting kubecfg for cluster
W0311 07:18:13.335336    1571 create_kubecfg.go:76] Did not find API endpoint for gossip hostname; may not be able to reach cluster
kops has set your kubectl context to vote.valaxy.net

Cluster is starting.  It should be ready in a few minutes.

Suggestions:
 * validate cluster: kops validate cluster
 * list nodes: kubectl get nodes --show-labels
 * ssh to the master: ssh -i ~/.ssh/id_rsa admin@api.vote.valaxy.net
 * the admin user is specific to Debian. If not using Debian please use the appropriate user based on your OS.
 * read about installing addons at: https://github.com/kubernetes/kops/blob/master/docs/operations/addons.md.
