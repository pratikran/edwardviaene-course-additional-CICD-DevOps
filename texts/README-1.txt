S5
L43
Jenkins: Autmoated Test

	Unit Test 
	Integration Test 
	Regression Test 
		for bug fix 
	UAT 

L44
Jenkins: Packaging
	java packages in .zip or .war (jar for java libraries)
	.deb on debian 
	.rpm on redhat 
	play framework 
		can package .deb 
	debian plugin 
	packaging code in build.sbt
	git commit and push 
	
L45
Jenkins: Automate testing and packaging Demo
	
	cd ci
	vagrant ssh 
	vim example-app/build.sbt 
	cd example-app/
	git commit -am "... "
	git push 
	
	Jenkins
		example-app
			build 
				Actions: test 
		Add Build Step 
			Execute shell 
				./activator debian/packagebin 
		Save and Build NOw
		check console log 
	
S6
L46
Deployment 
	
	
L47
Artifact storage and deployment
	
	create repository 
	
	sudo su - jenkins
	gpg --gen-key
	
	sudo apt-get install rng-tools
	sudo rngd -r /dev/urandom
	
	create signing key id, public key id
	
	repository configuration
	gpg --list-keys
	
	send public key to ubuntu key server to retrieve anytime later
		gpg --keyserver keyserver.ubuntu.com --send-key 0E6FF9F7
		
	repository 
		install nginx, reprepro
			then create repository
			configure jenkins to be able to write packages in the repository folder
			configure nginx, reprepro
			
	Post build actions to Jenkins
		add build step
			execute shell 
		
	build again
	
	
L48
Jenkins: Artifact storage and deployment Demo 
	vagrant ssh Jenkins 
	sudo su - jenkins
	gpg --gen-key 
		answer prompts 
	sudo su - vagrant 
	sudo apt-get install rng-tools 
	sudo rngd -r /dev/urandom 
	sudo su - jenkins 
	gpg --list-keysgpg
	gpg --keyserver keyserver.ubuntu.com --send-key 15DA5147
	sudo apt-get install nginx reprepro
	sudo -s
	mkdir /var/repositories
	cd /var/repositories
	mkdir conf 
	cd conf 
	touch options distributions
	vim distributions
		Codename: trusty
		COmponents: main
		Architecture: i386 amd64
		SignWith: 15DA5147
	chown -R jenkins:jenkins /var/repositories
	vim /etc/nginx/sites-available/default
		server { ...
		
	service nginx restart
	Jenkins server 
		job/example-app/configure
			Add Build Step 
				Execute Shell
					reprepro -b /var/repositories includedeb trusty _/target/example-app/*.deb
			Save and run build again 
	exit 
	cd ../chef
	vagrant up chef
	vagrant up node
	vagrant up workstation
	
	If host pc memory and resource is issue 
		may restart jenkins machine with lesser resources/memory 
		may stop workstation and run knife command on chef-server 
		
	vagrant ssh workstation
		cp /vagrant/chef-demo/recipes/app_deploy.rb cookbooks/my_cookbook/recipes
		vim cookbooks/my_cookbook/recipes/app_deploy.rb
			change the apt_repository key 
		cd cookbooks 
		vim my_cookbook/metadata.rb 
			increase version 
			add depends java 
		cp /vagrant/chef-demo/misc/example-app.json .
		BERKSHELF_PATH=~ berks install 
			installs my_cookbook in cookbooks dir starting in home dir 
			it fails 
		So,
			cd my_cookbook
			BERKSHELF_PATH=~ berks init
			BERKSHELF_PATH=~ berks install 
		cd ..
		knife cookbook upload apt 
		knife cookbook upload java 
		knife cookbook upload my_cookbook
		knife role from file example-app.json
		
		knife node run_list set node 'role[example-app]'
		ssh node 'sudo chef-client'
			(it should install java and deploy the app)
			(it may fail if workstation and jenkins machine IP addresses same, change vangratfile and restart)
		vagrant halt workstation
		vagrant up workstation
		vagrant ssh node 
			vim /etc/hosts 
				edit IP 
		vagrant ssh workstation
			vim /etc/hosts 
				edit IP 
			ifconfig
		ssh node 'sudo chef-client'
		ssh node 
			ps aux | grep example-app
		browse 
			http://192.168.0.3:9000
			(your new application is running)
			
L49
Continuous Monitoring 
	
	Stateless App 
		12factor app 
			12factor.net 
		
		Database data loss is saved 
		
	SDLC
							Plan			Monitor
	Developer 	>	Build 		>	Test 		>		Release 	>		Provision		>	Customer 
					Git				Unit									Deploy 
					Compile			Integration								Monitoring 
									Regression 
									UAT 
									More 
	
	Monitoring 
		Automation 
			is Continuous Monitoring
		
		Nagios
			1999 initial 
			plugins
			agents
			Nagios is retty old may be for cloud 
		Alternatives of Nagios
			Sensu
				modern and good for cloud 
			zabbix
			AWS Cloudwatch 
			
			
	Application Performance Monitoring 
		First Line of Monitoring 
			instant alerts when app slows down 
		Hosted Solution 
			New Relic 
		Application specific 
		
	Log File Aggregation 
		Log Files can give good insight 
		ELK(ELastic Search/Logstash/Kibana) can scale over 1000s of servers
		Its more work to setup 
		Complex algorithm can be run 
			Intrusion detection and cybersecurity
		Visulizations gives goood insights 
	
	
L50
Twelve Factor App 
	
	SaaS
	
	12-Factor 
		Methodology 
			for SaaS 
		
		declarative setup for automation 
		Maximum portatbility
		suitable for modern cloud Platforms 
		Minimize divergence b/w dev and prod 
		can scale 
		
	Advantages of 12-Factor
		Decouple App 4m infrastructure/OS
		Enables Green/Blue environment deployments
			Green Environment runs 
				if any problem spin up blue environment 
					if alls well 
						bring down green environment 
						
		Zero downtime and upgrades 
		Good with CI/CD 
		Best Practice 
		
		Avoids software erosion 
			
	Deployment 
		Git > Bucket > New Instance (deploy and dependencies up) 
			add to LB 
				upgrade or shutdown old instance 
	
L51
12 Factors 
	1. codebase 
		Version control 
			eg git repo 
	2. Isolate or explicit dependencies 
		dependecnies manifest 
			nodejs - packges.json 
			java - maven.repositories
			etc 
	3. Config 
		Store Environment Variables
		manage dev/prod/staging/QA environment separate from codebase 
		creadentials should not go in version control
		
	4. Backing Services
		like DB 
			treated as attached resource 
			never hardcode but couple using config 
	5. Build, Release, Run 
		Strictly separate build and run stages 
	6. Processes 
		Execute app as 1 or more stateless processes 
	7. Port Binding 
		Export Services via Port Binding 
		totally self-contained, no runtime injection of webserver into execution environment 
			app itself opens a port to handle the request 
	8. Concurrency 
		Scale via Proccess Model 
			scale processes 
			process types seaparation using queue systems 
	9. Disposability 
		Start/Stop at moment notice 
		fast scaling, rapid/robust deployment
	10. Dev/Prod Parity
		Keep Dev/Staging/prod as similar as possible 
			Minimize CI/CD gap, minimize tool gaps 
	11. Logs 
		Treat logs as streams, apps should output logs to stdout 
			It should not write or manage log files 
	12. Admin Processes 
		Run admin/management tasks as one-off processes 
			should be run in identical environment to log running app processes 
			should run against release of app codebase 
				must ship with codebase to avoid loss of syncronization
				

S9
Containerization
				
L52
Introduction of Microservices

	diff to Monolithic App example 
	
	many smaller services
	
	app distributed 
	Horizontal scaling 
	VMs are often too slow, inefficient and bigger 
		containerization is better choice 
		
L53
Containerization; Docker 
	Docker Engine 
	Docker HUb 
	Docker uses LXC 
	
	Using boot2docker 
		linux 
	Vagrant with linux 
	Cloud 
		AWS ECS 
		GC GCE
		
	vagrant init ubuntu/trusty64
	vagrant up
	vagrant ssh 
	
	sudo apt-get install docker.io
	sudo docker run centos:7 /bin/echo 'Hello World'
	sudo docker run -i -t ubuntu:1404 /bin/bash 
	sudo docker run -p 127.0.0.1:8080:80 -i ubuntu:1404 nc -kl 80
	
	Dockerfile 
		commands a user calls to assemble a docker image 
		several commands in succession 
		
	nodejs 
		index.js 
		packages.json 
		
	Dockerfile
		
	docker build .
	
	
L54
Docker Demo 

	https://github.com/wardviaene/docker-demo
	
	Vagrantfile 
	vagrant ssh 
	
	sudo apt-get update && apt-get install docker.io 
	
	makdir docker-demo && cd docker-demo 
	cp /vagrant/docker-demo/index.js .
	cp /vagrant/docker-demo/package.json .
	cp /vagrant/docker-demo/Dockerfile .
	
	cat Dockerfile 
	cat index.js 
	
	sudo docker build .
	
	sudo docker run -p 8080:3000 -t <containerID>
	
	browser on host pc 
		http://<VangrantIP>:8080
		
L55
Docker Architecture
	
	VMs vs COntainers 
	
	Docker written in go programming language 
	
	Namespaces 
		every container has its own namespaces
			pid, net, ipc, mnt, uts(unique time sharing system)
	Control Groups
		Limitation and privatization of resources 
			CPU, Memory, CPU etc 
			
L56
Docker Images
	
	Docker Images and Layers 
	Docker images references 
		list of layers 
	Images can share layers
	COntainers read these layers as single image using storage driver 
	
	New container	
		Read/Write Layer 
			writable 4 container or user of container 
		Image layers are read only 
	
		Images layers can be shared
		write on container writable layer 
			is not shared 
			
		when stopped all container layer data is lost 
		
		docker commit 
			to add container layer to image 
			
	docker < 1.10
		UUID for layers 
	docker > 1.10
		hash for layers 
			better security 
			no ID collisions
			data integrity
			better sharing of layers b/w images 
			
		Layers
			better reuse of layers
			even if images are not from same build
		
	Copy on write
		processes with same data 
			will share data 
		one process changes data 
			data will be copied first and then changed 
			
		both images, containers use it 
			optimize disk use 
			performance of container start 
			
		Multiple container safely share underlying layer 
		
		
		
L57
Docker Volumes 

	Persistent data strategy 
	
	Volumes 
		special dir in container 
		bypasses container read/write thin layer 
		bypasses read only image layers 
		
		initializaed when containers start 
		shared by containers 
		
		change in data directly goes to host OS 
			not saved on image layers 
			on container removal volumes remains
				persistent store 
		
		create Volume
			
			sudo run --name example -v /data/webapp:/webapp --t -i ubuntu:1404 /bin/bash 
			
		Volumes Plugins 
			when host system is removed 
				need to tranfer volume data beforehand 
				
			Plugins 
				help to integrate docker engine with external storage systems
			
				underlying data can be traferred to where container is running 
					independent of host OS 
					
				Flocker 
					data on external storage
					plug Flocker 
					Flocker agent map to external storage 
					
					
L58
Docker Networking 
	
	Bridged network by default 
		difficult to maintain 
	Better way 
		User Defined Network 
		
		Containers within same user defined network can communicate 
			but cant communicate with those outside 
			
		User Defined Bridge Network (Single Host)
		Overlay Network (Multi Host)
		
	create new bridge network 
		docker network create --driver bridge my_network 
		
		launch container 
			docker run --net=my_network -itd --name=container1 ubuntu:1404 
			docker run --net=my_network -itd --name=container2 ubuntu:1404 
			
			docker run --net=my_network -itd -p 80:80 --name=container3 ubuntu:1404 
				external container will have a port 80 open on container3 
				
	Overlay Network 
		Multi-Hosts 
		Key-Value store 
			start on one node, evetually on 3/5 nodes 
			
			consul, etcd, Zookeeper etc 
			
		create 
			docker network create --driver overlay my_overlay_network 
			
		Launch container 
			docker run --net=my_overlay_network -td --name=container1 ubuntu:1404 
			
			docker engine will store configuration dtails to key-value store 
			
L59
Docker Hub 
	Public and Private Registry 
	
	Alternative 
		AWS docker registry 
		Self-Hosted 
		
	Docker client 
	Docker Host 
	Docker Hub or Registry 
	
L60
Docker Compose 

	Multi-Container environment as a Code 
	
L61
Docker Compose Demo 
	
	vagrant ssh docker 
	
	docker --version 
		> 1.10 
		
		(sudo apt-get upgrade or apt-get install docker.io)
		
	sudo -s
		docker compose install command can be found on docker compose site , run here 
		
	chmod +x /usr/local/bin/docker-compose
	
	usermod -G docker vagrant 
	exit
	vagrant ssh docker 
	
	git clone https://github.com/wardviaene/docker-demo
	
	cd docker-demo
	cat docker-compose.yml 
	vim index-db.js 
	
	docker-compose build && docker-compose up db 
	
	vagrant ssh docker 
	cd docker-demo 
	docker-compose up -d web 
		(-d for bakground process)
	curl http://localhost:3000
	
	docker ps 
	
	docker-compose down 
	
L62
Docker Machine 
	
	Enable you to provision docker engine on VM 
		
	It can manage hosts 
		docker client and docker daemon 
	It can provision docker hosts on remote 
		eg AWS cloud 
	It provision docker host 
		with virtualbox on physical PC 
	
	Docker Toolbox 
		run docker without using vagrant 
		
		available for windows/MAC 
			comes with DOcker - engine, compose, Machine 
			includes kinematic 	
				GUI to manage containers 
				
L63
Docker Machine Demo 

	AWS 
		vangrant ssh docker 
		sudo -s 
		follow and run docker machine wbsite for command to install it 
		exit
		docker-machine 
		docker-machine ls
		
		set AWS configuration config 
		set docker-machine create command 
			run the command
		
			docker-machine create --driver amazonec2 --amoazonec2-region=eu-west-1 --amazonec2-vpd-id=vpc-43382923 -amazonec2-subnet-id=subnet-2q34355 --amazonec2-zone c aws02 
			
		docker-machine env aws02 
		eval $(docker-machine env aws02)
		docker ps 
		
		docker run -d -p 3000:3000 nodejs-demo 
		
		docker ps 
		docker-machien ip aws02 
		curl IP:3000
		
		docker ps 
		docker logs <containerID>
		
L64
Docker Swarm
	
	CLuster containers natively 
	
	swarm image is to be pulled and used 
	
	docker-machine can be used to provision hosts and install swarm 
	discovery backend can also be enabled when using a key-value store 
		consul, etcd, zookeeper
	
	Architecture 
		Manager Node 
			containers 
				consul
				swarm master 
		agent1
			container
				swarm 
			new containers using swarm 
		agent2
			container 
				swarm 
			new containers using swarm 
			
	vagrantfile
		start 3 nodes 
	Or
	docker toolbox and docker machine 
	
	git clone https://github.com/wardviaene/docker-swarm-demo
				
	vagrant up manager 
	vagrant up agent2 
	vagrant up agent3 
	
	vagrantfile
		docker_install.sh shell script 
		
	install swarm container on manager node 
		vagrant ssh manager
			docker run swarm --help 
		
		Multiple ways for discovery service on nodes 
			token 
				using docker hub discovery service and Unique ID ie discovery Token 
					suitable for dev and test 
			discovery backend 
				etcd, consul, zookeeper 
					good for production 
					
			1 consul container in dev but atleast 3 in production 
			
		start consul on master 
			docker run -d -p 8500:8500 --name=consul progrium/consul -server -bootstrap 
		
		start swarm manage node on master 
			docker run -d -p 4000:4000 swarm manage -H :4000 --advertise 192.168.0.248:4000 consul://192.168.0.248:8500 
			
		On agents 
			run swarm join to let them join cluster 
			
			docker run -d swarm join --advertise=192.168.0.248:2375 consul://192.168.0.248:8500 
			
		check on master 
			docker -H :4000 info 
			
		to run container on swarm 
			run docker run 
				thru swarm 
			docker -H :4000 run -p 3000:3000 -d wardviaene/nodejs-demo 
			
		docker -H : 4000 ps 
		
		curl 192.168.0.247:3000 
		
L65
Docker Swarm Demo 
	
	git clone https://github.com/wardviaene/docker-swarm-demo
	
	cp docker-swarm-demo/Vagrantfile .
	
	vagrant up manage agent2 agent3
	
	
	
S10
Container Orchestration


L66
INtroduction to Container Orchestration

	Built in 
		Docker Swarm 
		Docker Compose 
	
	Container Managers (orchestrators)
		Mesos 
			was available before docker 
		Kubernetes 
	Platform as a Service 
		Heroku 
		Dockku & Dies 
		Flynn 
		
	
L67
Kubernetes Architecture OverView 

	OPen Source 
	
	
L69
Deploying contiainers using Kubernetes 

	vagrant ssh k8s
	
	vim /etc/default/grub
		GRUB_CMDLINE_LINUX="cgroup_enable=memory swapaccount=1"
		
	sudo update-grub 
	
	sudo reboot 
	
	
L70
Kubernetes Demo
	

S11	
L71
DevOps Challenge 
	
	www.devopschallenge.co 
	
	
	
		
			
		
	
	
		
	
		
	
		
	
			
		
	
	
	
		
		
	
		
		
	