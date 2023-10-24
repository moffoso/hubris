### Hardening SSH access
Check what file is overriding /etc/ssh/ssh_config
```bash
sshd -T
# In my case the Output was: /etc/ssh/sshd_config.d/50-cloud-init.conf
```

>[!warning]
> ```/etc/ssh/sshd_config.d/50-cloud-init.conf ```
> is a symlink for cloud.init
> https://cloudinit.readthedocs.io/en/21.1/topics/faq.html

add following lines to the file
```bash
PasswordAuthentication no
PermitRootLogin no
ChallengeResponseAuthentication no
UsePAM no
```

test the config with -o PubkeyAuthentication=no
```bash
ssh master@10.130.5.130 -o PubkeyAuthentication=no
# Output will be this:
# master@10.130.5.130: Permission denied (publickey).
```

### Install and configure kubectl
The installation of kubectl is analogue as with the server.
The configuration works as follows:
1. copy the config on the master node
```bash
cat ~/.kube/config
```

2. create the config in the client and paste the content
```bash
sudo mkdir -p $HOME/.kube
cat <<EOF | sudo tee $HOME/.kube/config
# insert your config here
EOF
```

configure the config to only be readable by your user
```bash
chmod go-r ~/.kube/config
```
### Install helm on the client
helm doesnt need to be configured as it uses the kubectl config
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### QoL fzf, ctx and ns
1. install fzf
```bash
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```
CTRL-T, CTRL-R, ALT-C

2. install krew (kubectl plugin manager)
```bash
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
```

3. append following line to .zshrc and restart shell
```bash
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
```

4. install ctx and ns with krew
```bash
kubectl krew install ctx
kubectl krew install ns 
```


