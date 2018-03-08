1. 使用Let's Encrypt的免费证书, 有效期3个月

2. 官方推荐的工具[certbot](https://certbot.eff.org), 按这个工具的官网教程自动化生成证书

   - 注意域名必须先指向这个服务器的ip
   - 打开服务器的80端口
   - 安装certbot后, 执行时可能会报`ImportError: No module named 'requests.packages.urllib3'`的错误, 这时需要执行下面的命令

     ```Shell
     pip uninstall requests
     pip uninstall urllib3
     yum remove python-urllib3
     yum remove python-requests
     yum install python-urllib3
     yum install python-requests
     yum install certbot
     ```

3. 编写脚本自动更新证书

   - 新建每周定期任务脚本
    
     ```Shell
     vim /etc/cron.weekly/update-letsencrypt.sh
     ```

   - 在文件中输入以下内容

     ```Shell
     #!/bin/bash
     
     /bin/certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
     ```
