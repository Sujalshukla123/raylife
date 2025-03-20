## Get Dump
1. pg_dump -n schema_name  -h production_database_host_name -p postgres_post -W -U user_name -d database_name > sql_file_name.sql (ex. pg_dump -n hub  -h rayprod.cnalrpjgvvuy.ap-south-1.rds.amazonaws.com -p 5432 -W -U RayApp -d RayApp > hub.sql)
2. enter password : kba5tuF3Ckd4H5DT
3. copy file to machine where postgresql deploy

## install postgre
1.  sudo yum install postgresql
2.  psql -U user_name -h postgres_deployed_machine_ip -p postgres_node_port  -d database_name < sql_file_name.sql (ex. psql -U RayApp -h 172.27.63.174 -p 32384  -d RayApp < hub.sql)