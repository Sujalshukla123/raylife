
ssh Baiju@172.27.63.175 -p 5522

# SetUP
## Install docker
1. sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
2. sudo dnf install docker-ce 
3. sudo systemctl start docker 
4. sudo systemctl enable docker 
5. docker --version

## Install Docker compose
1. sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose 
2. sudo chmod +x /usr/local/bin/docker-compose 
3. vi ~/.bashrc
4. export PATH=$PATH:/usr/local/bin (Add this line at the end of the above file) 
5. source ~/.bashrc 
6. docker-compose --version

## Create folders and Copy Files
1. mkdir config 
2. mkdir docker-compose 
3. cd config 
4. vi logstash.conf 
5. Copy folder config and file (in repo path : elk/config/*)
6. Copy folder docker-compose and file (in repo path : elk/docker-compose)

## Run
1. cd ~/docker-compose
2. docker-compose up -d

## Permission (if docker service fail to start for permission issue then execute below commands)
1. sudo chmod 777 logstash.conf
2. sudo chmod -R 775 /data

## Firewall Allow 5000 port for communication
1. sudo firewall-cmd --zone=public --add-port=5000/tcp --permanent 
2. sudo firewall-cmd --reload

## Configure UI
1. In browser open http://172.27.63.175:5601/app/discover#/
2. In UI there is option to "create data view" click on it.
3. In dialog box fill value as below.
    Name: logs
    Index pattern: logs-docker-compose
4. Click on sava data view to kibana
