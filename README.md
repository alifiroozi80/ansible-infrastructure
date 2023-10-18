This repository contains a couple of Ansible PlayBooks that deploy and bring on the below resources on Ubuntu and CentOS (Detection is automatic):

1) [Kubernetes](https://kubernetes.io) Cluster
    * One Master, multiple workers (Test)
    * Multiple Master, Multiple Workers (Production)
2) A [Teleport](https://goteleport.com) Instance
3) A [HA-Proxy](https://www.haproxy.com) (Mandatory for Production K8s setup!)
4) A [Mattermost](https://mattermost.com) (Deploy/Backup)
5) A [Jenkins](https://www.jenkins.io) Agent
6) And soon, more resources! (See the below Roadmap)

---

## Run through VPN?

Also, at the beginning of each playbook, it will ask you whether you want to set up a VPN on the target host or not.
It's helpful if you want the installation to go through a VPN.

Why?

For various reasons, for instance, you live in sanctions countries like Iran.

If you want the installation to go through the VPN, you must already have an OpenConnect VPN server. (Do you want to deploy an OpenConnect VPN server? See `Setup OpenVPN on a server` below)
Ansible runs the VPN with the `openconnect` client on the machine.
And pass the IP, username, and Password to Ansible whenever it asks you.

<!-- RUNNING -->

## Running playbooks

First, Install roles dependcies:

```shell
$ ansible-galaxy install -r requirements.yaml
```

---

To run a specific play:

```shell
$ cd playbooks
$ ansible-playbook -i ../inventory.ini <PLAYBOOK>.yaml
```

* NOTE: Everything is already set. You only need to change the IP address and your ssh key in the `host_vars` or `group_vars`

---

At times, certain things are encoded. To execute such playbooks, follow these steps: 

1. Go to the `playbooks` directory.

2. Run the command
```shell
ansible-playbook --ask-vault-pass -i ../inventory.ini <PLAYBOOK>.yaml
```

<!-- Notes on K8s Playbook -->

### Notes on K8s Playbook

As I mentioned earlier, you have two options for K8s:

1) Single master node (and multiple worker nodes):
This is pretty straightforward.
Just put the master IP in the `[control-plane]` section.
And that's it.

2) Multiple Master nodes (and multiple worker nodes):
    * We are using the [Stacked ETCD](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#stacked-etcd-topology) model
    * As you know, in this model, you must have a load balancer for your API servers (See [Here](https://github.com/kubernetes/kubeadm/blob/main/docs/ha-considerations.md#options-for-software-load-balancing))
    * We are using HAProxy for our HA K8s setup.
    * Notice you can use this HAproxy for communication with your cluster as well, but keep in mind that mainly in a Production environment, you should separate the API Server's load balancers from other load balancers.
    * One last note: **just put one of your master IPs in `[control-plane]` and the rest goes under `[masters]`**

<!-- Notes on Teleport Playbook -->

### Notes on Teleport Playbook

Remember to change the `proxy_service.public_addr` and `proxy_service.acme.email` in the Teleport config file in `roles/teleport/files/teleport.yaml`

<!-- INVENTORY -->

## About Inventory

* There is always exactly **ONE** IP under `ha-proxy` and `control-plane`.
* If you want to run `k8s-single.yaml` play, the `masters` should be exactly **ONE** IP.
* If your K8s Cluster is behind a bastion (Jump Host), then be sure to add the below line to the `group_vars/GROUP`.

```yaml
ansible_ssh_common_args: '-o ProxyCommand="ssh -i KEY -p 22 -W %h:%p -q USER@BASTION-IP"'
```

## Setup OpenVPN on a server

It's super easy. You need a Server, A Public IP, and Docker.

We use [this](https://github.com/TommyLau/docker-ocserv) image.

Run the below command on the server that you want to be your VPN server.

```shell
docker run --name ocserv --privileged -p 443:443 -p 443:443/udp -d tommylau/ocserv
```

Then, create a username and Password for your self (Here is my username, `ali`, put yours instead of ali)

```shell
docker exec -ti ocserv ocpasswd -c /etc/ocserv/ocpasswd -g "Route,All" ali
```

<!-- ROADMAP -->

## Roadmap

- [x] Add [Kubernetes](https://kubernetes.io)
- [x] Add [Teleport](https://goteleport.com)
- [x] Add [HA-Proxy](https://www.haproxy.com)
- [x] Add [Mattermost](https://mattermost.com)
- [x] Add [Jenkins](https://www.jenkins.io)
- [ ] [Installation of Jenkins Controller](https://www.jenkins.io/doc/book/installing)
- [ ] [Installation of Mattermost](https://docs.mattermost.com/install/install-rhel-6-mattermost.html)
- [ ] Deploying [LDAP](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)
- [ ] Deploying [KeyCloak](https://www.keycloak.org)

See the [open issues](https://github.com/alifiroozi80/ansible-infrastructure/issues) for a complete list of proposed features (and known
issues).

<!-- CONTRIBUTING -->

## Contributing

Any contributions you make are **greatly appreciated**.

If you have a suggestion to improve this, please fork the repo and create a pull request. You can also open an issue
with the tag "enhancement."

1) Fork the Project
2) Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3) Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4) Push to the Branch (`git push origin feature/AmazingFeature`)
5) Open a Pull Request

<!-- LICENSE -->

## License

The license is under the MIT License. See [LICENSE](https://github.com/alifiroozi80/ansible-infrastructure/blob/main/LICENSE) for more
information.

---

## ❤ Show your support

Give a ⭐️ if this project helped you!
