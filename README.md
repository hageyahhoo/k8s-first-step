# k8s-first-step

## あとで清書する
Dockerでの動作確認
"docker container run -d -p 81:80 --name <container_name> nginx:latest”のように、ポートを明示的に指定する必要あり。
・portの設定
　・前者：from：アクセス時のport
　・後者：to：Container内のport
　・”docker port <container_name>” で、"80/tcp -> 0.0.0.0:81” のように確認可能
　・この設定を行わないとアクセス不可。
・この設定後、http://localhost:81 でアクセス可能。
・Containerの起動と共に、nginxも起動している。
　・"nginx -s stop”を実行すると、Containerも停止する。


https://hub.docker.com/_/nginx/
・/bin
　・ある：vi・less・ps
　・ない：cat・more・bash
・nginxの操作方法：nginx
・/var/log/nginx
　・access.log
　・error.log
・/etc/nginx/conf.d/default.conf
