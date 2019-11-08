# Image Classifier and Comparator

Purpose: Set up a static webpage and server to host classifier/pairwise comparator tasks for images 

I am not the original author of these files. This repository contains aggregated and updated files from previous developers. See acknowledgements below.

## Pre-Install Instructions

1. Set up AWS instance. A free tier server is adequate for small scale deployment. I also recommend setting up an SFTP client such as FileZilla for easier file transfer (will require your public IPv4/DNS and a security key .pem file)
2. SSH to your server using your custom command/security key provided by AWS.

## Instructions for setup

### Initial setup

> Make sure you are in your home directory *~/home/ubuntu* or *~/home/\<user>* if on a local machine

1. ```git clone https://github.com/QTIM-Lab/Image-Comparator.git```  

2. ```cd Image-Comparator/Image-Comparator``` 

3. Install these libraries if your system doesn't already have them (more necesary for docker containers as they are light weight with minimal installed software).

> AWS VMs have all but ruby

```bash
apt-get install -y git
apt-get install -y python
apt-get install -y ruby
apt-get install -y vim
apt-get install -y curl
```

4. Deploy server using the following:

*python2*  
```bash
$ python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

*python3*
```bash
$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

> Without the use of *tmux* or *screen* your server will stop when the terminal is terminated. 

To use screen perform the following:

```bash
$ screen -S http_server
```

Once in the screen run ```python3 -m http.server 8080``` and then use *cntrl+a, d* to exit screen leaving it running. Finally use ```screen -ls``` to see available screens:

```bash
$ screen -ls
There is a screen on:
	13545.http_server	(11/08/19 20:28:20)	(Detached)
1 Socket in /run/screen/S-ubuntu.
```

Now anytime you need to get into the screen to stop the server run ```screen -r http_server```, and you will be dropped back into the screen.


## Instructions for setup via **Docker**. **NOT A NEED**
3. Create docker image using instructions described here: https://github.com/CollinWen/Image-Comparator-Dockerfile
4. Of note, the above repository's dockerfile works, but does not work completely, and some of the webpage code did not work. I have attached my proposed fixes in my dockerfile but have not been able to verify if it works. If the dockerfile above does not work for you, I would recommend replacing the above repository's files with this repository's files and using sed to replace all of my IP addrresses with your server's.

...TDB

### Couchdb Setup

To install couchdb, run the following:
```
echo "deb https://apache.bintray.com/couchdb-deb bionic main" \ | tee -a /etc/apt/sources.list
curl -L https://couchdb.apache.org/repo/bintray-pubkey.asc \ | apt-key add -
apt-get update && apt-get install couchdb
```

Use the following configurations:
* 1: single node
* 0.0.0.0 bind address
* (password)

To ensure that the database is accessible locally, check the following locations to make sure that all instances of bind_address are set to '0.0.0.0' (127.0.0.1 means that the database is only accessible on the host machine)
* /opt/couchdb/etc/local.ini
* /opt/couchdb/etc/default.ini
* /opt/couchdb/etc/default.d/10-bind-address.ini

Next confirm that CORS (cross-origin resource sharing) is enabled in the following locations
* /opt/couchdb/etc/default.ini
    * enable_cors = true
    * under '[cors]', add the following:
	* origins = *
	* credentials = true
	* methods = GET, PUT, POST, HEAD, DELETE
	* headers = accept, authorization, content-type, origin, referer, x-csrf-token
* /opt/couchdb/etc/local.ini
    * under '[httpd]', add the following:
	* enable_cors = true
    * at the bottom, add:
	* [cors]
	* origins = *
	* credentials = true
	* methods = GET, PUT, POST, HEAD, DELETE

Once you've confirmed the bind_address and CORS are configured properly, you must restart the couchdb server by using the following command:
```
service couchdb restart
```

To finish configuring a single node setup, run the following;
```
curl -X PUT http://admin:<password>@localhost:5984/_users
curl -X PUT http://admin:<password>@localhost:5984/_replicator
```

### Test Connections

If you have opened ports 8080 (for the http site) and 5984 (couchdb) then you should be able to navigate to these urls:

```
http://\<aws-ip-address--or--dns>:8080
http://\<aws-ip-address--or--dns>:5984
```

or

```
http://\<local-ip-address>:8080
http://\<local-ip-address>:5984
```

or

```
http://\<localhost>:8080
http://\<localhost>:5984
```

## Instructions for use
1. To create and setup the database, run the following (use the <db_name> you used from the docker build command):  

```bash
$ curl -X PUT http://admin:<password>@localhost:5984/<db_name>
$ cd /Image-Comparator/dbutil
$ curl -X PUT http://admin:<password>@localhost:5984/<db_name>/_design/basic_views -d @basic_views.json
```

2. Add Images to DB:

```bash
$ ruby addImagesToDb_jkc.rb <imageFolder> <imageSetName>
```

* <imageFolder> is ```/data``` inside the docker, which was mounted during container creation.  
* <imageSetName> is the name for the image set.  

> \<imageFolder> can be any folder if not using docker.


3. Add images to an Image Compare List:

```bash
$ makeICLFromImageSetName.rb <image set name> <pct repeat> <list name>
```
* \<image set name> was the name given to addImagesToDb_jkc.rb  
* \<pct repeat> is the percentage of repeated pairs to be displayed  
* \<list name> is a new list name  

4. Add a task to a user:

```bash
$ makeTask.rb <user> <image-list name> <image-list-type> <task-order>
```
* \<user> is who should complete the task  
* \<image-list name> this is <list name> from above in (makeICLFromImageSetName.rb)  
* \<image-list-type> ```compare``` or ```OCTcompare```.  
* \<task-order> is what precedence the image compare task takes relative to others assigned to this user.   

5. Change Image-Comparator/ui/two_image.html to have your users in this section:

```html
<!-- manually add users that you assigned tasks too in makeTasks.rb -->
<select id="username" onchange="OnSetUser(this.value)">
  <option selected="selected">Choose your user</option>
  <option value="grader1">Grader 1</option>
  <option value="grader2">Grader 2</option>
  <option value="grader3">Grader 3</option>
  <option value="grader4">Grader 4</option>
</select>
```

## Acknowledgements

In the order they appear this project has been forked and added to. Yet again we fork to make a QTIM-Lab based project that will be adapted and maintained here.

1. Jayashree Kalpathy-Cramer, PhD for original source code (https://github.com/AlanCramer/Image-Comparator)  
2. Collin Wen (https://github.com/CollinWen/Image-Comparator.git)  
3. Collin Wen (https://github.com/CollinWen/Image-Comparator-Dockerfile)  
4. Jimmy chen (https://github.com/jche253/Fundus_Classifier_Comparator)  


## Contact
For questions, please contact:
* Benjamin Bearce, bbearce@gmail.com  
* Jayashree Kalpathy-Cramer, PhD kalpathy@nmr.mgh.harvard.edu  
