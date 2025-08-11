# Sunbeam-Contribution
Notes on how to setup a machine and contribute to the Sunbeam project

## Setup

### User housekeeping

You will need:-
- github.com account
- launchpad account
- to sign the CLA https://ubuntu.com/legal/contributors#sign-the-contributor-agreement


### Fresh ubuntu install

```bash
lxd init --auto

LATEST_STRICT_MICROK8S=$(snap info microk8s | grep strict | grep -v tracking | head -1 | awk '{ print $1 }')
snap install microk8s --channel $LATEST_STRICT_MICROK8S

sudo usermod -a -G snap_microk8s $USER
sudo chown -R $USER ~/.kube
newgrp snap_microk8s

microk8s status (should be running)
microk8s kubectl get pods -A (calico and coredns should be running ok)

sudo microk8s enable hostpath-storage
juju bootstrap microk8s
```

You will need an editor, VS Code is not bad at all

```bash
snap install code --classic
```

Tox and Skopeo will be needed for later

```bash
sudo apt install -y tox skopeo
```

Docker is handy for Rock creation

```bash
sudo snap install docker
sudo groupadd docker
sudo usermod -a -G docker $USER
sudo chown root:docker /var/run/docker.sock
newgrp docker
```


### Git setup

```git config -l```

Ensure you setup your user email and name:-

```
user.email=someone@somewhere.com
user.name=Some Body
```

Additionally we need to add some settings for gpg signing. If you don't already have a gpg key, set one up as per https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key

```
user.signingkey=1234ABCD1234ABCD
commit.gpgsign=true
tag.gpgsign=true
```

Confirm your key as per https://docs.github.com/en/authentication/managing-commit-signature-verification/checking-for-existing-gpg-keys



### Git copies

Fork the following:-

https://github.com/canonical/ubuntu-openstack-rocks - it should then show in your github e.g https://github.com/mv-2112/ubuntu-openstack-rocks





### Github commits

1. Ensure you have a signing key setup on your github account, follow the instructions here https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key

2. Ensure git knows about your signing key, https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key#telling-git-about-your-gpg-key

3. You are now ready to fork the repository at https://github.com/canonical/snap-openstack

4. cd into the directory you have just created and create a new branch ```git branch -m "feat/new shiny"```

5. Make your changes

6. Check your changes and git add as required

7. Commit your changes, ```git commit -S -m "feat/my new shiny"``` (the -S should be superfluous)

8. If you have made more than one commit, you should squash them, ```git rebase -i HEAD~2``` where 2 is the number of commits you need to merge

9. Check with ```git log --show-signature``` that you are about to only push 1 signed commit to upstream

10. Finally, ```git push```


## Rocks

https://github.com/canonical/ubuntu-openstack-rocks/

Creating rocks is fairly simple, these are the base images that we use to bring up a service.

- For uid's refer to https://github.com/electrocucaracha/openstack-multinode/blob/master/etc/kolla/kolla-build.ini

you may need to identify where a product comes from, e.g Trove in universe https://launchpad.net/ubuntu/+source/openstack-trove, vs Cinder in main https://launchpad.net/ubuntu/+source/cinder

follow the README.md to pack and test

```bash
sudo snap install rockcraft --edge --classic
rockcraft pack
```

You can now upload the containers to a docker registry.

```
skopeo login docker.io -u <user> -p <password>

ROCK_NAME=$(basename $(pwd))
ROCK_ARCHIVE=$(ls *${ROCK_NAME}*.rock)
ROCK_VERSION=$(yq ".version" ./rockcraft.yaml)
skopeo copy oci-archive:${ROCK_ARCHIVE} docker://verranm/${ROCK_NAME}:${ROCK_VERSION}
```

If your Rock contains a dashboard, you need to consider if it functions without that service (i.e. Horizon will get deployed with it regardless of if the service you are adding is enabled). It is likely a better plan to add it to the ```override-stage``` of the Horizon ```rockcraft.yaml``` file.

You can obtain the files for the dashboard with the following:-

```bash
curl -skL $(apt-get download -o Dir::Cache::archives="./" --print-uris python3-cloudkitty-dashboard/noble | awk -F\' '{print $2}' ) | dpkg-deb -c /dev/stdin | grep "openstack_dashboard" | grep -v "^d" | awk '{ print $6 }' | xargs -I{} basename {}
```



## Charms 

https://opendev.org/openstack/sunbeam-charms



