# Hadoop Provisioner
This project is able to do the provisioning of a multi-node environment with hadoop/spark.

### Tecnology
This script was made with Ansible. To do local tests (before running it on homolog and production environments), is better to create local virtual machines with Vagrant.

This tutorial assumes you are running ubuntu on your host.

### Tips to do local tests before running the script on production environment

**Step 1: creating virtual machines**

- make sure you have public and private keys in your machine. To do it, type `ls ~/.ssh` and verify the files `id_rsa` and `id_rsa.pub` exist.
- in your project, type `vim vm/master/Vagrantfile` and change the line
    `ssh_pub_key = File.readlines("/home/zem/.ssh/id_rsa.pub").first.strip`
    for
    `ssh_pub_key = File.readlines("/home/YOUR_USERNAME/.ssh/id_rsa.pub").first.strip`. 
    If you don't know your username, type `whoami`.
- `cd vm/master`
- `vagrant up --provider virtualbox`
- when the prompt asks for the network interface you ant to use, type 1
- repeat these steps for node1 and node2 

The previous step have created 3 virtual machines with CentOS 7. To verify they are running fine, type `vagrant global-status`.

**Passo 2: Inventory**

In this step, you will check for the ip of each node and set a inventory file. To do it, follow these steps:

- `vagrant global-status`
- take note of the master id (something like **316ea25**)
- type `vagrant ssh <<master_id>>`, the same id you just took note.
- type `sudo su` to check you change to root user. Type `exit`
- type `ip addr` and take the note of the `eth1` interface ip.
- type `exit` to come back to your machine.
- repeat these steps to `node1` e `node2`.

Now, type `vim ansible/inventory_local`.
- Below `[hadoop_master]`, type the ip of the master node.
- Below `[hadoop_workers]`, type the ip of each worker node.
- Save the file.

**Passo 3: Befor running the script**

- Before running the ansible script, assure you can access all the server nodes without using a password. For each inventory ip, type `ssh vagrant@server_ip`. If you can access the server, fine. If you need to type some password to access the server, find information about how to copy public ssh key to the server.
- The user `vagrant` need to be able to be root. To check it, you need to execute this command: `sudo su`.
- Edit the files `playbook_master.yml` and `playbook_slaves.yml`. Replace all words `zem` for your username.

**Passo 4: Running the script**

- Enter your project root, and type the following commands:
- `cd ansible`
- `ansible-playbook -i inventory_local playbook_master.yml --extra-vars "ansible_sudo_pass=asdf"` - first time it fails
  `TASK [se esta habilitado o ipv6, reboota]`
- wait for 1 minute or two
- `ansible-playbook -i inventory_local playbook_master.yml --extra-vars "ansible_sudo_pass=asdf"` - second time, it succeeds
- `ansible-playbook -i inventory_local playbook_slaves.yml --extra-vars "ansible_sudo_pass=asdf"` - it works for the first time

**Passo 5: Formatting namenode**

Verify each node of the server, with the user `hduser` can access the other nodes:
- `ssh vagrant@ip_master`
- `sudo su - hduser`
- `ssh hduser@localhost`
- `exit`
- `ssh hduser@master`
- `exit`
- `ssh hduser@slave1`
- `exit`
- `ssh hduser@slave2`
- `exit`

Now, you need to format the namenode. Type this command on the master node: `hdfs namenode -format`.

**Passo 6: Start the services**
- On your master node, type these commands:
  - `start-all.sh`  
  - `cd $SPARK_HOME`
  - `cd sbin`
  - `./start-all.sh`

**Passo 7: It is ready!**
- Check if all the processes are running with `jps` commandd in each node. The output of this command are some lines like `master`, `worker`, `jps`, `` 
- Check if there are active nodes on yarn: `yarn node -list -all`. It is expected two nodes running.

# Possible improvments
- Remove warnings when running spark and hadoop
- Avoid rebooting the machine when  running the script
- Reuse code: playbook_master and playbook_slaves are almost the same

# References
- `http://backtobazics.com/big-data/setup-multi-node-hadoop-2-6-0-cluster-with-yarn/`
