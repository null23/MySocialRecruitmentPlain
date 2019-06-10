upstream test_corp_001 { 
    server 10.32.4.23:8080; 
    server 10.24.5.123:8080; 
    server 10.52.1.532:8080; 
    server 10.61.6.29:8080; 
    hash $remote_addr; 
} 

location /etl/ { 
proxy_pass http://test_corp_001/; 
proxy_set_header Host $host; 
proxy_set_header X-Real-Scheme $scheme; 
proxy_set_header X-Real-IP $remote_addr; 
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
} 

注意proxy_pass http://test_corp_001/，后边如果带/，就会把配置的etl这个url缀删除。如果后边不带/，请求的url就会带着etl。