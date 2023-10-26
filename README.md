# Kubernetes CRD and Custom Controllers

In Kubernetes, you can think of a "Custom Resource" as a way to create your own special objects to manage things that are unique to your application. These special objects are like the standard things Kubernetes knows how to handle, like Pods and Services, but they are tailored to your specific needs.

Imagine you're running a video game on Kubernetes, and you want to create a custom resource to represent a new type of in-game item. You can define the properties of this item, like its name, power, and special abilities, using something called a "Custom Resource Definition" (CRD).

A Custom Controller is a software component you create to watch and manage Custom Resources. It can automate tasks, like scaling applications or handling custom logic, based on the state of these custom objects.

# Current Environment

- Go (1.18)
- Kubectl (GitCommit : 1b4df30b3, Git Version: v1.27.0)
- KubeBuilder (3.4.0)
- Packages need to Install using apt such as make, build-essential
- Linux/AMD64

# Establish Current Environment
- Open Terminal.
- Install Packages such as make, build-essential and go 1.18.
```
controlplane ~ ➜ sudo apt update && sudo apt install make build-essential 
controlplane ~ ➜  export VERSION=1.18
controlplane ~ ➜  curl  -L https://golang.org/dl/go${VERSION}.linux-amd64.tar.gz -o go${VERSION} && tar -xzf go${VERSION} -C /usr/local
controlplane ~ ➜  export PATH=$PATH:/usr/local/go/bin
controlplane ~ ➜  go version
go version go1.18 linux/amd64
```
- Now Configure Your Git.
```
controlplane ~ ➜  git --version
git version 2.25.1
controlplane ~ ➜  git config --global user.email EMAILID
controlplane ~ ➜  git config --global user.name USERNAME
controlplane ~ ➜  git config --global credential.helper   cache
controlplane ~ ➜  
```
- Install KubeBuilder 3.4.0
```
controlplane ~ ➜  cat > kube_builder.sh
curl -L https://github.com/kubernetes-sigs/kubebuilder/releases/download/v3.4.0/kubebuilder_linux_amd64 -o kubebuilder_3.4.0_linux_amd64
chmod +x  kubebuilder_3.4.0_linux_amd64
sudo mv  kubebuilder_3.4.0_linux_amd64 /usr/local/bin/kubebuilder
^C
controlplane ~ ➜  bash kube_builder.sh
```

# Create an Empty Go Module
- Create CRD Directory
```
controlplane ~ ➜  mkdir CRD
controlplane ~ ➜  cd CRD
```
- Specify the location of all the project dependencies in GOMODCACHE.
```
controlplane ~/CRD ➜  go env -w GOMODCACHE=$PWD/.deps/pkg/mod
controlplane ~/CRD ➜  go env | grep GOMODCACHE
GOMODCACHE="/root/CRD/.deps/pkg/mod"
```
- Initialize this whole directory as Go Module and check the contents.
```
controlplane ~/CRD ➜  go mod init github.com/USERNAME/BookCRD
go: creating new go.mod: module github.com/USERNAME/BookCRD
controlplane ~/CRD via 🐹 v1.18 ➜  ls -la
total 20
drwxr-xr-x 3 root root 4096 Oct 26 02:14 .
drwx------ 1 root root 4096 Oct 26 02:12 ..
drwxr-xr-x 3 root root 4096 Oct 26 02:14 .deps
-rw-r--r-- 1 root root   50 Oct 26 02:14 go.mod
controlplane ~/CRD via 🐹 v1.18 ➜ 
```
# Push This change to the Github Repo.
- Create Github Repo with name 'BookCRD' [https://github.com/new] with desired visibility.
- Go Back to Terminal 
- Initialized CRD as Git Repo
```
controlplane ~ ➜  cd CRD
controlplane ~/CRD via 🐹 v1.18 ➜  git init
Initialized empty Git repository in /root/CRD/.git/
```
- Add README.md file and Save it.
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  vi README.md
```
- View the Content  
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  git status
```
- Mention '.deps' in .gitignore. Whatever you mention in .gitignore, it will not be pushed to repository.
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  vi .gitignore
```
- Now if you try the below command, you will not see .deps and there is no need to worry becuase .deps can be re-generated via 'go mod download'.
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  git status
```
- Push the Content to Staging Area.
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  git add .
```
- View the Staged Changes.
```
controlplane CRD on  master [?] via 🐹 v1.18 ➜  git status
```
- Now Commit so that later it can be pushed to Github. The Output will be similar as shown below.
```
controlplane CRD on  master [+] via 🐹 v1.18 ➜  git commit -m "first commit"
[master (root-commit) 155261f] first commit
 2 files changed, 4 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 go.mod
```
- Define Remote Repository URL. 
```
controlplane CRD on  master via 🐹 v1.18 ➜  git remote add origin https://github.com/Aditya98Shukla/BookCRD.git
controlplane CRD on  master via 🐹 v1.18 ➜ git branch -M main
```
- Push the Main branch to Remote Repo. Make Sure you have mention Gihub USERNAME and Personal Access Token as Password.
```
controlplane CRD on  master via 🐹 v1.18 ➜  git push -u origin main
```
## Initializing Kubernetes Project
- Clone the Repository
```
controlplane ~ ➜  git clone https://github.com/Aditya98Shukla/BookCRD.git
Cloning into 'BookCRD'...
remote: Enumerating objects: 5, done.
remote: Counting objects: 100% (5/5), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 5 (delta 0), reused 5 (delta 0), pack-reused 0
Unpacking objects: 100% (5/5), 2.18 KiB | 2.18 MiB/s, done.
controlplane ~ ➜  cd BookCRD
```
- Check If there are any changes. I hope not.
```
controlplane BookCRD on  main via 🐹 v1.18 ➜  git status
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```
-  Initializes a Kubernetes project using the Kubebuilder framework and sets the domain for your project to "genesis.xyz.com." This domain typically represents the root domain for your Kubernetes Custom Resource Definitions (CRDs) and can help identify your custom resources in a cluster.
```
controlplane CRD on  main via 🐹 v1.18 ➜  kubebuilder init --domain genesis.xyz.com
Writing kustomize manifests for you to edit...
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.11.2
......
go: downloading github.com/nxadm/tail v1.4.8
go: downloading gopkg.in/tomb.v1 v1.0.0-20141024135613-dd632973f1e7
go: downloading github.com/cespare/xxhash v1.1.0
Next: define a resource with:
$ kubebuilder create api
```
- Note the KubeBuilder Version.
```
controlplane CRD on  main [!?] via 🐹 v1.18 ➜  kubebuilder version
main.version{KubeBuilderVersion:"3.4.0", KubernetesVendor:"1.23.5", GitCommit:"75241ab9ff9457de77e902645792cee41ba29fed", BuildDate:"2022-04-28T17:09:31Z", GoOs:"linux", GoArch:"amd64"}
```
- You will notice the changes after applying 'git status'. Move these changes to staging area using 'git add .' and then commit. Later Push the committed changes.

## Creating CRD, Custom Resource and Custom Controller

- In this step, create a new API for the custom resource. We'll create an API named Book in the compute group and version v2.
```
controlplane BookCRD on  main [!?] via 🐹 v1.18 ➜  kubebuilder create api --group comp --version v2 --kind Book
Create Resource [y/n]
y
Create Controller [y/n]
y
Writing kustomize manifests for you to edit...
.......
```
- The Above command generates various files, including the CRD definition, controller, and API type definition.

- Implement your new API and generate the manifests (e.g. CRDs,CRs) with:
```
controlplane BookCRD on  main [⇡] via 🐹 v1.18 ➜  make manifests
```
- You will notice that a CRD YAML is created at the location 'config/crd/bases' and its name is 'comp.genesis.xyz.com_books.yaml'.
```
controlplane BookCRD on  main [?] via 🐹 v1.18 ➜  ls config/crd/bases
comp.genesis.xyz.com_books.yaml
```
