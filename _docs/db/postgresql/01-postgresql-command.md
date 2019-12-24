---
title: "linux 계열 명령어 정리"
permalink: /docs/os/linux/linux-command/
excerpt: "linux command list"
last_modified_at: 2019-07-18T15:53:52-04:00
redirect_from:
  - /theme-setup/
toc: true
---

### 

0. docker exec -it {container_id} /bin/bash

1. postgres로 유저 변경
   ``` bash
   root$ su - postgres
   ```

2. 
   ``` 
   postgres$ psql -c "CREATE DATABASE mapjoyserver OWNER kepri ENCODING 'utf-8';"
   ```



3. 
   ```bash 
   postgres$ psql -d mapjoyserver -f mapjoyserver_DDL.sql
   ```




   /mapjoy/service/2/mms