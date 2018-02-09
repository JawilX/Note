1. 使用Let's Encrypt的免费证书, 有效期3个月
2. 官方推荐的工具[certbo](https://certbot.eff.org), 按这个工具的官网教程自动化生成证书, 注意域名必须先指向这个服务器的ip
3. 编写脚本自动更新证书
    - 新建每周定期任务脚本
    ```
    vim /etc/cron.weekly/update-letsencrypt.sh
    ```
    - 在文件中输入以下内容
    ```
    #!/bin/bash

    /bin/certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
    ```
