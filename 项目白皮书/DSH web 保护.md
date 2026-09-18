**1.看有没有干活**
```
cat /var/log/dsh-watchdog.log     # 有内容说明重启过，空文件说明一直健康
```

**2. 存一份应急信息**（万一哪天打不开，按这个查）

复制
```
# 服务是否活着
systemctl is-active dsh-web

# 隧道是否在跑
ps -ef | grep "[c]loudflared"

# 服务挂了就重启
systemctl restart dsh-web

# 看实时日志
journalctl -u dsh-web -f

```

## 3.**要配对/加设备**

```powershell
ssh -L 3099:127.0.0.1:3080 root@120.24.109.194
```

**出现 `Welcome to Alibaba Cloud ECS !` 就成功了 —— 这个窗口别关**
打开
```
http://127.0.0.1:3099/
```
如果弹出 401，说明登录令牌过期了，去服务器取新的：
如果弹出 401，说明登录令牌过期了，去服务器取新的：

bash

复制
```
journalctl -u dsh-web --no-pager | grep -oE "/\?token=[A-Za-z0-9._-]+" | tail -1 | sed 's|/?token=||'
```
然后把令牌拼上去打开：
```
https://127.0.0.1:3099/token=
```




