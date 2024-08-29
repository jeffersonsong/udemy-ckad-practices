1. Identify the name of the Operating system installed.

```shell
cat /etc/os-release 
PRETTY_NAME="Ubuntu 22.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.4 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

2. Install the helm package.
If unsure how to install the helm tool, feel free to refer to the documentation. The Documentation tab is available at the top right panel.

- helm installed?

[Installing Helm](https://helm.sh/docs/intro/install/)

```shell
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
Helm v3.15.4 is available. Changing from version v3.15.3.
Downloading https://get.helm.sh/helm-v3.15.4-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

helm --help
```

3. Use the help page of helm command to identify the command used to retrieve helm client environment information.

env

```shell
helm --help | less
  env         helm client environment information
```

4. Identify the version of helm installed on the cluster.

3.15.4

```shell
helm version
version.BuildInfo{Version:"v3.15.4", GitCommit:"fa9efb07d9d8debbb4306d72af76a383895aa8c4", GitTreeState:"clean", GoVersion:"go1.22.6"}
```

5. What is a command line flag used to enable verbose output?

--debug

6. That’s all for now. That was a quick introduction to the helm command line utility. Feel free to explore the helm command line utility further. We will learn more about these commands throughout the remainder of this course.
