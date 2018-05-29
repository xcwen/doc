# 使用 openresty (nginx+lua)  管理   页面权限

在验证用户的网页  生成cookie 

登录时间: login_checktime

登录账号: login_username

md5 验证信息: login_md5sum   算法:  login_checktime + "|"+  login_username +"|"+ "[约定码]";





将 实际的服务 用   openresty (nginx) 反向代理

然后 在 openresty 检查  login_md5sum, 登录事件

```config
    upstream resinserver {  
        server  192.168.0.2:8080;
    }  

    server {
        listen       80;
        server_name  www.kks.com;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
		        default_type text/html;
		        access_by_lua_block {
			          login_checktime=ngx.var["cookie_login_checktime"];
			          login_username=ngx.var["cookie_login_username"];
			          login_md5sum=ngx.var["cookie_login_md5sum"];
			          now= os.time();
			          if tonumber(login_checktime) >  now +120     then 
				        ngx.say("error login checkt time" ) ;
				        return ngx.exit(200);
			          end
			          if (tonumber(login_checktime) <  now -3600    ) then 
				        ngx.say(" need re login  :<a href=\"http://www.xxx.com\">admin.leo1v1.com </a>"  ) ;
				        return ngx.exit(200);
			          end
		            md5_str=login_username..":"..login_checktime..":md5keyXXX";

			          if ngx.md5(md5_str )  ~= login_md5sum   then 
				        ngx.say(" check md5 error,  need re login  :<a href=\"http://www.xxx.com\">www.xxx.com </a>"  ) ;
				        return ngx.exit(200);
			          end	
		        }
     		    proxy_pass   http://resinserver;  
	      }
    }
```

