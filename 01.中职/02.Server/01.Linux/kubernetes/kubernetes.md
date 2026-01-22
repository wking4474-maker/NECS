## 题目:
- 在linux5-linux7上安装containerd和kubernetes，linux5作为master node，linux6和linux7作为work node；使用containerd.sock作为容器runtime-endpoint。导入nginx镜像，主页内容为“HelloKubernetes”。
- master节点配置calico，作为网络组件。
- 创建一个deployment，名称为web，副本数为2；创建一个服务，类型为nodeport，名称为web，映射本机80端口和443端口分别到容器的80端口和443端口。

## 解:
```shell    
openssl genpkey -algorithm RSA -out server.key
openssl req -new -key server.key -out server.csr
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

kubectl create secret tls my-secret --cert=server.crt --key=server.key
```


```yaml
# https configmap https.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  http.conf: |
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
            add_header Content-Type text/plain;
            return 200 'HelloKubernetes\n';
        }
    }
  https.conf: |
    server {
        listen       443 ssl;
        server_name  localhost;
        ssl_certificate /etc/nginx/ssl/tls.crt;
        ssl_certificate_key /etc/nginx/ssl/tls.key;
        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
            add_header Content-Type text/plain;
            return 200 'HelloKubernetes\n';
        }
    }
```

```yaml
# deployment deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: docker.io/library/nginx:9.9
        ports:
        - containerPort: 80
        - containerPort: 443
        volumeMounts:
        - name: config-volume
          mountPath: /etc/nginx/conf.d
        - name: config-ssl
          mountPath: /etc/nginx/ssl
      volumes:
      - name: config-volume
        configMap:
          name: nginx-config
          items:
          - key: http.conf
            path: http.conf
          - key: https.conf
            path: https.conf
      - name: config-ssl
        secret:
          secretName: my-secret
```


```yaml
# service service.yaml

apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    nodePort: 30080
  - name: https
    port: 443
    targetPort: 443
    nodePort: 30443
  selector:
    app: web

# port 表示 Service 对外暴露的端口号，targetPort 表示服务后端 Pod 中容器实际监听的端口号。
```
