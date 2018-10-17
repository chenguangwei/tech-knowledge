nginx
========

nginx.conf 配置

- [nginx配置详情（总结）](https://www.jianshu.com/p/6e5c9095e350)

- [Nginx静态服务配置---详解root和alias指令](https://www.jianshu.com/p/4be0d5882ec5)



root 指令

location /dir/ 
root root_path ->  http://host/dir/file.txt  -> root_path/dir/file.txt
alias 指令

location /dir
alias alias_path ->  http://host /dir /file.txt  -> alias_path/file.txt

location /dir/ 
alias alias_path/ ->  http://host /dir/ file.txt  -> alias_path/file.txt

