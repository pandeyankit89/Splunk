# Splunk Enterprise Setup on AWS EC2 (Index Cluster + Search Head Cluster)

### Step 1 : Created account in splunk.com, which allows to use splunk for 6 months for free.

### Step 2 : Login to AWS. "EC2" -> Security Group
- (2.1) Created a Security Group with name `sg-splunk` with Inbound-Rule as -
    - (a) allow inbound traffic from "0.0.0.0/0 (Anywhere)" to Web-Port (8080/TCP) and SSH (22/TCP)
    - (b) allow inbound traffic from "VPC subnet" to Management-Port(8089), KV-store- Port (8191/TCP), Replication-Port for Index Cluster(9000/TCP), Replication-Port for Search-Head-Cluster (9100/TCP)
      ![security-group for splunk](/img/sg-splunk.png)
	- (2.2) Created 7 ubuntu EC2 instances of available free-tier "t3.small" with 15 GB Storage and security-group as `sg-splunk` and a .pvk file to login through putty.

### Step 3: Login to each ec2 instance and install Splunk Enterprise - 
```javascript
# Deafult user -
login as: ubuntu

# Update packages -
sudo apt update && sudo apt upgrade -y

# Download Splunk Enterprise from URL and untar -
cd /opt
sudo wget -O splunk.tgz "https://download.splunk.com/products/splunk/releases/9.4.3/linux/splunk-9.4.3-237ebbd22314-linux-amd64.tgz"
sudo tar xvzf splunk.tgz

# After untar and unzip, delete .tgz file to clean some space -
sudo rm splunk.tgz

# Create a user `splunk` and make it owner of /opt/splunk directory -
sudo useradd -r -m -s /bin/bash splunk
sudo chown -R splunk:splunk /opt/splunk

# switch to splunk user 
sudo su - splunk
cd /opt/splunk/bin

# Install Splunk Enterprise -
/opt/splunk/bin/splunk start --accept-license

# Provide administrator user and its password -
Please enter an administrator username: admin
Please enter a new password: admin@123
Please confirm new password: admin@123

# Once Installed and Started, check the Status -
/opt/splunk/bin/splunk status
```

### Step 4: Prepare Splunk Web URL :
Go to AWS => EC2 => Instance => check each instance one by one => Copy Instance `Public IP` => modify Splunk URL like http://<public IP>:8000/ => open in Browser

### Step 5: Change minimum free disk limit : 
Go to Settings => Server Settings => Change the minimum free disk space limit 5000 MB to 1000. => go to `Server Control` => Restart Splunk Instance [Do this for each instance, as in POC we will have less space] 

### Step 6: Setup Approach :

### Index-Cluster Tier :
	
- (6.1) Index-Tier consists of a "Manager-Node", earlier called "Cluster Master Node" and Other indexers as "Peer Nodes"
    - (a) To install Index-Cluster, First step to setup  as "Manager-Node". Login as admin to URL http://<public IP>:8000/  of manager-node => Setting => Index Clustering => select `Enable Index Clustering` => select "Manager Node" =>  It needs - 
	      - (i) replicationfactor 
	      - (ii) search factor
	      - (iii) security-key
	      - (iv) Index-Cluster name 
    - (b) restart splunk => /opt/splunk/bin/splunk restart
    - (c) check /opt/splunk/etc/system/local/server.conf
    - (d) Login to each Peer Nodes with their respective URLs http://<public IP>:8000/ =>  Setting => Index Clustering => select `Enable Index Clustering` => select "Peer Node" =>  It needs -
          - (i) Manager server URL => http://<Private IP of Manager-Node>:8000/
				  - (ii) Replication port = 9000 (as earlier decided and mentioned in AWS security-Group)
				  - (iii) Security-Key = same what was mentioned in Manager-Node
		- (e) restart splunk on each instance => /opt/splunk/bin/splunk restart
		- (f) check /opt/splunk/etc/system/local/server.conf  on each instance.
				
		- (g) Login to Manager Node URL  	http://<public IP>:8000/ => Settings => check `Index Clustering` => It should show as below -
		
		
- Helpful URLs :		
    - (1) Enable the manager node => https://help.splunk.com/en/splunk-enterprise/administer/manage-indexers-and-indexer-clusters/9.4/deploy-the-indexer-cluster/enable-the-indexer-cluster-manager-node#id_38e88f88_6b73_46d2_b74e_ca687d057eff__Enable_the_indexer_cluster_manager_node
    - (2)  Enable the peer => https://help.splunk.com/en/splunk-enterprise/administer/manage-indexers-and-indexer-clusters/9.4/deploy-the-indexer-cluster/enable-the-peer-nodes#eb28970b_f065_4aa8_8735_013338a9aa58__Enable_the_peer_nodes

- Note :
    - (1) If want to create a new index => create on Manager-Node
    - (2) If want to send data => send to any Peer node

### Search-Head-Cluster Tier :
	
- (6.2) Search-Head-Cluster consists of a `Deployer` and multiple `Search-Heads`. One among them will be configured as `Captain` search-head. Deplyoer is responsible for deploying `Apps` on each seacrch-heads.
	Note : Deployer functionality is automatically enabled on all Splunk Enterprise instances. The main configuration step is to specify the deployer's security key.  Later in the deployment process, you point the cluster members at this deployer instance, so that they have access to it. 
		
    - (a) Configure the deployer's security key => deployer uses the security key to authenticate communication with the cluster members. The cluster members also use it to authenticate with each other.  => To set the key on the deployer, specify the pass4SymmKey attribute in the [shclustering] stanza of the deployer's server.conf file -
```javascript
# Stop Splunk -
/opt/splunk/bin/splunk stop

# Add  shclustering stanza in server.conf -
vi /opt/splunk/etc/system/local/server.conf

[shclustering]
mode = deployer
pass4SymmKey = SearchHeadSecret123
shcluster_label = sh_cluster

# Start Splunk -
/opt/splunk/bin/splunk start

# Check server.conf -
more /opt/splunk/etc/system/local/server.conf
```

- Note : Three members, so that the cluster can continue to function if one member goes down. Search head clustering supports up to 50 members in a single cluster. defaults to 3.
- Caution: Always use new instances. The process of adding an instance to a search head cluster overwrites any configurations or apps currently resident on the instance.
- Important: You must change the admin password on each instance. The CLI commands that you use to configure the cluster will not operate on instances with the default password.


    - (b) Creating Search Heads : Run the splunk init shcluster-config command and restart the instance: [This command is only for cluster members. Do not run this command on the deployer. + You can only execute this command on an instance that is up and running.]

```javascript
splunk init shcluster-config -auth <username>:<password> -mgmt_uri <Search-Head-1 Https URL>:<management_port> -replication_port <replication_port> -replication_factor <n> -conf_deploy_fetch_url <Deployre Https URL>:<management_port> -secret <security_key> -shcluster_label <label>

/opt/splunk/bin/splunk restart
```
```javascript
# Login to Search-Head-1, switch to splunk user:

login as: ubuntu
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk init shcluster-config -auth admin:admin@123 -mgmt_uri https://<Search-Head-1 EC2 Private IP>:8089 -replication_port 9100 -replication_factor 2 -conf_deploy_fetch_url https://<deployer EC2 Private IP>:8089 -secret SearchHeadSecret123 -shcluster_label sh_cluster

/opt/splunk/bin/splunk restart
more /opt/splunk/etc/system/local/server.conf


# Login to Search-Head-2, switch to splunk user:

login as: ubuntu
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk init shcluster-config -auth admin:admin@123 -mgmt_uri https://<Search-Head-2 EC2 Private IP>:8089 -replication_port 9100 -replication_factor 2 -conf_deploy_fetch_url https://<deployer EC2 Private IP>:8089 -secret SearchHeadSecret123 -shcluster_label sh_cluster

/opt/splunk/bin/splunk restart
more /opt/splunk/etc/system/local/server.conf
```
    - (c) Select Captain =>  Run the splunk bootstrap shcluster-captain command on the selected instance:
```javascript
splunk bootstrap shcluster-captain -servers_list "<URI>:<management_port>,<URI>:<management_port>,..." -auth <username>:<password>
``
```javascript
# Login to Search-Head-1, switch to splunk user:

login as: ubuntu
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk bootstrap shcluster-captain -servers_list "https://<Search-Head-1 EC2 Private IP>:8089,https://<Search-Head-2 EC2 Private IP>:8089" -auth admin:admin@123

# Restart Splunk -
/opt/splunk/bin/splunk restart

# check server.conf -
more /opt/splunk/etc/system/local/server.conf

# Command to check Search-Head-Cluster Status -
/opt/splunk/bin/splunk show shcluster-status --auth admin:admin@123
```
---
- Important: The URIs that you specify in -servers_list must be exactly the same as the ones that you specified earlier when you initialized each member, in the -mgmt_uri parameter. You cannot, for example, use https://foo.example.com:8089 during initialization and https://foo.subdomain.example.com:8089 here, even if they resolve to the same node.

    - (d) Connect Search-Head Cluster to Index-Cluster : configure each member of the search head cluster as a search head on the indexer cluster. Search heads get their list of search peers from the manager node of the indexer cluster. Configure each search head cluster member as a search head on the indexer cluster. 

      - Note :
        - (1) The secret key that you set here is the indexer cluster secret key (which is stored in pass4SymmKey under the [clustering] stanza of server.conf), not the search head cluster secret key (which is stored in pass4SymmKey under the [shclustering] stanza of server.conf).
        - (2) You must run this CLI command on each member of the search head cluster.

```javascript
splunk edit cluster-config -mode searchhead -manager_uri https://<Indexer Manager-Node EC2 Private IP>:8089 -secret <key while confgiruing Index-Cluster> -auth <login of this search head>:<password of this search head> 

splunk restart
```
```javascript 
# Login to Search-Head-1, switch to splunk user and run cluster-config command :
login as: ubuntu
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk edit cluster-config -mode searchhead -manager_uri https://<Indexer Manager-Node EC2 Private IP>:8089 -secret <key while confgiruing Index-Cluster> -auth admin:admin@123

# Restart Splunk :
/opt/splunk/bin/splunk restart

# check server.conf :
more /opt/splunk/etc/system/local/server.conf

# Login to Search-Head-2, switch to splunk user and run cluster-config command :
login as: ubuntu
sudo su - splunk
cd /opt/splunk/bin
/opt/splunk/bin/splunk edit cluster-config -mode searchhead -manager_uri https://<Indexer Manager-Node EC2 Private IP>:8089 -secret test@123 -auth admin:admin@123

# Restart Splunk :
/opt/splunk/bin/splunk restart

# check server.conf :
more /opt/splunk/etc/system/local/server.conf
```
    - (e) Distribute Configurations/App via Deployer to Search-Head:

```javascript
# Login to Deployer, switch to splunk user :
login as: ubuntu
sudo su - splunk

# Create an application and conf files inside it :
cd /opt/splunk/etc/shcluster/apps
mkdir splunk_my_first_app
cd splunk_my_first_app
mkdir default
cd default
vi inputs.conf  => Test => :wq!

# Apply this Bundle to Search-Head Captain. Run on Deployer with deployer creadentials  :

/opt/splunk/bin/splunk apply shcluster-bundle -target https://<EC2 Private IP of Search-Head-1 as it is Captain>:8089 -auth admin:admin@123

# Verify the bundle is applied on all search heads.

# Login to Search-Head-1, switch to splunk user and check for the app directoty and conf files :
login as: ubuntu
sudo su - splunk
cd /opt/splunk/etc/shcluster/apps
ls -l
cd splunk_my_first_app
more inputs.conf

# Login to Search-Head-2, switch to splunk user and check for the app directoty and conf files :

login as: ubuntu
sudo su - splunk
cd /opt/splunk/etc/shcluster/apps
ls -l
cd splunk_my_first_app
more inputs.conf
```

  - (6.3) Imporatnts Commands :
  -  (i)  To check the overall status of your search head cluster, run this command from any search-head :
    
```javascript
splunk show shcluster-status -auth <username>:<password>
```
    (ii) To check the status of the KV store running on the cluster. Run this command from any search-head :
    
```javascript
splunk show kvstore-status -auth <username>:<password>
```
    (iii)  To check the status of Index-Cluster, Run this command on manager-Node :

```javascript
/opt/splunk/bin/splunk show cluster-status -auth admin:admin@123
```
---

