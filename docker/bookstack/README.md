**How to backup SQL database**

>docker exec bookstack_db mysqldump -u root -p(password) bookstackapp > data.sql  
>docker stop bookstack  
>docker stop bookstack_db  
>tar -czvf bookstack_backup_DATE.tar.gz data.sql bookstack_app_data bookstack_db_data -v
