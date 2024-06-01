Notes :  

Make sure both containers are in the same docker network (default is called bookstack_default), otherwise you might see this issue in bookstack app container logs
> nc: getaddrinfo for host "bookstack_db" port 3306: Name does not resolve

**How to backup SQL database**

>docker exec bookstack_db mysqldump -u root -p(password) bookstackapp > data.sql  
>docker stop bookstack  
>docker stop bookstack_db  
>tar -czvf bookstack_backup_DATE.tar.gz data.sql bookstack_app_data bookstack_db_data -v
