# Setup k8s Environment

## Task

> Setting up our Linux environment to make it easier to work with k8s.

## Preliminary Steps

1. Check the architecture map at https://www.lucidchart.com/documents/view/58e22de2-c446-4b49-ae0f-db79a3318e97/0_0
2. Check server details at https://kodekloudhub.github.io/kodekloud-engineer/docs/projects/nautilus

## Research

* Bash completion - https://kubernetes.io/docs/tasks/tools/included/optional-kubectl-configs-bash-linux/

## Steps

```bash
# Install Bash completion.
sudo -i
yum install -y bash-completion
```

```
...
Installed:
  bash-completion.noarch 1:2.1-8.el7
...
```

```bash
# Log out or manually source Bash completion.
source /etc/profile.d/bash_completion.sh

# Set up k8s bash auto completion.
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'alias k=kubectl' >>~/.bashrc
echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
source ~/.bashrc

cat >> ~/.vimrc
```

```
set nu
set list
set hls
set expandtab
set shiftwidth=2
set tabstop=2
```

Close the file with control + d i.e. `^D`

Our environment has now been set up.