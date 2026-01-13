# nacos
How to use and install nacos

## nacos error: server error : errCode: 500, errMsg: do metadata operation failed,下线失败

```
# 进入 Nacos 安装目录
cd /path/to/nacos
# 删除 protocol 文件夹
rm -rf data/protocol
```

## nacos-mysql
```
apiVersion: apps/v1
kind: Deployment
#kind: ReplicationController
metadata:
  name: mysql
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      initContainers:
      - name: init-schema
        image: busybox:1.35  # 或 alpine/curl
        command: ["/bin/sh", "-c"]
        args:
        - |
          NACOS_VERSION=3.1.1
          SCHEMA_URL="https://raw.githubusercontent.com/alibaba/nacos/${NACOS_VERSION}/distribution/conf/mysql-schema.sql"
          wget -O /initdb/mysql-schema.sql "$SCHEMA_URL" || exit 1
          echo "Schema downloaded"
        volumeMounts:
        - name: initdb
          mountPath: /initdb
      containers:
      - name: mysql
        image: mysql:8.0.30
        args:
        - "--character-set-server=utf8mb4"
        - "--collation-server=utf8mb4_unicode_ci"
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
        - name: MYSQL_DATABASE
          value: "nacos_devtest"
        - name: MYSQL_USER
          value: "nacos"
        - name: MYSQL_PASSWORD
          value: "nacos"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
        - name: initdb
          mountPath: /docker-entrypoint-initdb.d
      volumes:
      - name: mysql-data
        persistentVolumeClaim:
          claimName: mysql-pvc  # 先创建 PVC
      - name: initdb
        emptyDir: {}  # 临时卷，只用于首次 init
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: gp3  # 或你的 EKS StorageClass
```
