
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
1. mkdir ray
2. cd ray
3. mkdir docker
4. cd docker
5. vi lighthouse.yml 
6. copy file content (in repo path /lighthouse/lighthouse.yml)

## Run
1. systemctl start docker
2. cd ~/ray/docker 
3. docker login
    username: truecomtelesoftindia
    password: coldplay121
3. docker-compose -f lighthouse.yml up -d

## Firewall Allow 5000 port for communication
1. sudo firewall-cmd --zone=public --add-port=38116/tcp --permanent 
2. sudo firewall-cmd --reload
