# README #

This README would normally document whatever steps are necessary to get your application up and running.

### What is this repository for? ###

* Quick summary
* Version
* [Learn Markdown](https://bitbucket.org/tutorials/markdowndemo)

### How do I get set up? ###

* Summary of set up
* Configuration
* Dependencies
* Database configuration
* How to run tests
* Deployment instructions

### Contribution guidelines ###

* Writing tests
* Code review
* Other guidelines

### Who do I talk to? ###

* Repo owner or admin
* Other community or team contact


# Create secret from onprem secret creation to pull from aws ecr

aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 582395889164.dkr.ecr.ap-south-1.amazonaws.com

kubectl create secret generic aws-ecr-pull --from-file=.dockerconfigjson=/root/.docker/config.json --type=kubernetes.io/dockerconfigjson

/root/.docker/config.json should contain creds


kubectl delete secret aws-ecr-pull

kubectl create secret generic aws-ecr-pull --from-file=.dockerconfigjson=/root/.docker/config.json --type=kubernetes.io/dockerconfigjson


# managing postgres

CREATE USER "RayApp" WITH PASSWORD 'pa8Gz+cLfAT4Yn3';
CREATE DATABASE "RayApp";
GRANT ALL PRIVILEGES ON DATABASE "RayApp" TO "RayApp";
ALTER DATABASE "RayApp" OWNER TO "RayApp";
ALTER USER "RayApp" WITH SUPERUSER;
# How to access postgres

postgres pod is running in postgres namespace

use `kns` command to switch namespace and then `kubectl get po` OR run `kubectl get po -n postgres`

exec into the pod using the command `kubectl exec -it postgres-postgresql-0 -n postgres`

once pod get started get base64Encoded passwords in secrets using below command
kubectl get secret postgres-postgresql -o yaml
there is key : postgres-password 
in this there is base64Encoded password is stored and to use as that decode it and use it as a postgres password.

current password is oHDiVFCeNg

`psql -U postgres`

## When setting up haproxy

the coredns ip is hardcoded in the config. 10.96.0.10:53. This needs to be set to the ip of coredns service.


## store database dump
1. pg_dump -h raystoreprod.cnalrpjgvvuy.ap-south-1.rds.amazonaws.com -p 5432 -W -U postgres -d raystore_prod >raystoreprod.sql
   password : x9ZYVpLeupcMp8C9
2. Create database "raystore_prod";
3. CREATE USER "raystore" WITH PASSWORD 'pa8Gz+cLfAT4Yn3';
4. CREATE USER "rdsadmin" WITH PASSWORD 'pa8Gz+cLfAT4Yn3'; 
5. kubectl cp raystoreprod.sql postgres-postgresql-0:/tmp/raystoreprod.sql 
6. psql -U postgres -h localhost -d raystore_prod < raystoreprod.sql
   password : KjRHHY8sXA
