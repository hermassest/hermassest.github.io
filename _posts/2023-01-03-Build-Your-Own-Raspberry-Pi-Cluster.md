---
published: true
---

Have you ever wondered how to combine multiple Raspberry Pi boards into one single super computer. In this post I will guide you step by step to accomplish the above goal.

## Requirements
	1. One or more Raspberry Pi boards.
	2. Raspbian OS or any other Linux distribution of your choice).

## Let’s get started

First of all, you need to write a Raspbian OS image on **micro-SD** card, and an image writer application. To do so follow these steps:

- Plug in the micro-SD into your computer.
- Lunch Raspberry Pi Image.
- Click on `Choose OS` then navigate to `Raspberry Pi OS Other` select the desired operating system from the menu (in our case Raspberry Pi OS light).
- Select the available storage device, which is our micro-SD.
- You may want to tweak the configuration of the boot variables, via the settings button.
- Once you have done, click the **write** button.
- After completion, go to the micro-SD directory and search for file called `cmdline.txt`, then open it for editing.
- Jump to the end of the line and append the following:
`cgroup_memory=1 cgroup_enable=memory ip=<your ip address>::<gateway>:<subnet mask>:<hostname>:eth0:off`, then save and close.
- Then search for a file labeled `config.txt`, and open it.
- Scroll to the bottom of the file and append the following:
`arm_64bit=1`, then save and close.
- Create a new file called `ssh`, to enable the secure shell connection.
`$ touch ssh`
- Unplug the micro-SD from your computer, and plug it back to the Raspberry Pi adapter, then connect your Ethernet, the power the board.
- Wait couple seconds, until the boot process finishes, check for connectivity by typing this command:
```
$ ping -c5 x.x.x.x
```
- Woo-hoo! You are almost done, now it is time to connect to our lovely Pi via secure shell, type the following command:
```
$ ssh username@ip address
```
Then accept the RSA footprint for the first time, then type the default password `raspberry`
- Elevate your credential to root, `sudo -i`, then type the password.
 
 ### Installing kubernitese.
 
Now we will install kubernatese, it is a container orchestration program that will help us in this post.

So let's get started

- Install kubernatese via:
```
$ curl -sfL https://get.k3s.io | k3s_KUBECONFIG_MODE="644" sh -s- 
```

### Adding some nodes to obay the master.
Now we will get the status of our raspberrys and  add multiple raspberry pies to serve our master machine.

- Get the status of the raspberry pies via:
```
$ kubectl get nodes
```

> the meaning of node is any hardwere containing softwere.

_Repeat the steps above if you have multiple raspberry pies in defrent terminals for each pi board._

- To add a raspberry pi to serve, we need the master token via:
```
cat /var/lib/rancher/k3s/server/node_token
```
- Now to regester our pi boards use the command bellow
```
$ curl -sfL - https://get.k3s.io | K3S_TOKEN="<your master token>" K3S_URL="https://<your master ip>:6443 K3S_NODE_NAME="<servername> sh -"
```
### Instaling rancher (optional but recomended).
In this section se will install rancher for visualized configuration.

- First you need a virtual machine appart from the pi boards you have, so intialise a vertual inviroment via any virtual machine programm (_recomended softwere: ubentu 18.04 4GB_).

- Use the scure shell via terminal to log to the vertual machine, then elevate your credential to root.
- So now you need to make two directoris vie the command bellow:
```
$ mkdir /etc/rancher
$ mkdir /etc/rancher/rke2
```
- Switch to the directory rke2 and make a file:
```
$ nano config.yaml
```
- Append to the file the following:
```
1. token: <personal token>
2. tls-san:
3.   - <ip of the server you are currently in>
```
- Now, you will install rancher via the following command:
```
$ curl -sfL https://get.rancher.io | sh -
```

- You will enable the rancher service via:
```
$ systemctl enable rancherd-server.service
```

- Enter the following command to start the srvice:
```
$ systemctl start rancherd-server.service
```

- So now, you will monitor what is happening via the following command:
```
$ jurnalctl -eu rancherd-service -f
```
> If you want to stop monitoring or any process press `ctrl + C `

- You will reset the password of rancher, so you need the default username and password with the following command:
```
$ rancherd reset-admin
```

- Access the rancher by taking the ip given by the last command into any search engine.
- Enter the default username and password.
- When the next page showes, reset the password.
- Before clicking continue make sure you select the first option bellow the reset panel, so you can edit your clusters.
- Agree on terms and conditions, then click continue.
- Then when the new window pop-up save the ip by clicking `save`.

### Adding your raspberry pies to our virtual server.

- Click on the `add cluster` button.
- Now choose the `other` option.
- Name your cluster and click on create.
- Copy the third command and paste it in the terminal (if the command does not exicute, run it again) then go back to rancher and click `done`.
- Now the clusters page should show.

- Click on the little dots on the left side of your cluster, then click on `show API`.
- Click on `edit`, and paste the following into the `agentImageOverride`
```
rancher/rancher-agent:v2.5.8-linux-arm64
```
- Scroll till the end and click the `show request` button, scroll till the end and click `send request`.
- Then click on `close` and close this tab.
- Click on `explore` on the left of your cluster.
- Yay! your server is added to the vertual machine.

### Installing your first application.
In this step you will install your first applicaton on your raspberry pi cluster.

- First thing first, go to the master node's terminal.
- Now you need a [manifest](https://medium.com/@yogitakothadia/a-manifest-file-in-kubernetes-952183a508d4) file that describes the application you want to install.
- Make sure the file extends with `.yaml`

- Then apply the manifest fiel by the command bellow:
```
$ kubectl apply -f <filename>.yaml
```
- To view the status of your application use the following command:
```
kubectl get pods
```
> A pod is a [container](https://www.docker.com/resources/what-container/)

- view wich raspberry pi is on via:
```
$ kubectl get nodes -o wide
```

Now if you have just installed your first application on your cluster, ok, but if you want to access it you can't.

Let's make sure we can access our app by the steps bellow.

- you need another manifest file, this is an example of an ngin-x web server:
```
apiVersion: v1
kind: Service
metadata:
  name: ngin-x_nodeport
  namespace: default
spec:
  type: NodePort
  selector:
  	app: ngin-x
  ports:
  	name: web
  	port: 80
  	targetport: web
  	nodeport: 31111
```
- Apply all of that stuff via:
```
$ kubectl apply -f <filename>.yaml
```
- You could show the services via:
```
$ kubectl get services
```

- Keep going, You are almost done!!!
 
### Using ingress with rancher.
Here, we will use ingress, which is a DNS that will point to an application on your server

- Go to your rancher page, then click on `ngresses`
- Click `create` then put the DNS you want in the `request host` inputbox.
- Put a `/` in the second input box.
- Choose the node you want to point to in the third drop down menu.
- Select the port from the last dropdown menu.
- Click on save.
-Search for a file called `posts`.
- Scroll till the end and add the following:
`
<the ip of any node> <your DNS >
`
- Save and close.

> CONGRATUALTIONS!! Your Server Is Compleated!!

> Please share with your family and friends!