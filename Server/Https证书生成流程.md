1. 使用Let's Encrypt的免费证书, 有效期3个月
2. 官方推荐的工具[certbo](https://certbot.eff.org), 按这个工具的官网教程自动化生成证书, 注意域名必须先指向这个服务器的ip
3. 编写脚本自动执行证书续期任务
   - 通过 systemd 来自动执行证书续期任务
     ```shell
     sudo vim /etc/systemd/system/letsencrypt.service
     ```
     ```shell
     [Unit]
     Description=Let's Encrypt renewal

     [Service]
     Type=oneshot  
     ExecStart=/usr/bin/certbot renew --quiet --agree-tos  
     ExecStartPost=/bin/systemctl reload nginx.service #使用nginx时需要
     ```
   - 增加一个 systemd timer 来触发这个服务
     ```shell
     sudo vim /etc/systemd/system/letsencrypt.timer
     ```
     ```shell
     [Unit]
     Description=Monthly renewal of Let's Encrypt's certificates

     [Timer]
     OnCalendar=daily
     Persistent=true

     [Install]
     WantedBy=timers.target
     ```
   - 启用服务，开启 timer
     ```shell
     sudo systemctl enable letsencrypt.timer
     ```
     ```shell
     sudo systemctl start letsencrypt.timer
     ```
