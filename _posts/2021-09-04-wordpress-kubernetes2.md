---
layout: post
title: Ejecutar Wordpress + MySQL en Kubernetes paso a paso 2, agregando persistencia
published: true
categories: k8s docker wordpress mysql tutorial
---

# Introducción

En el [post anterior](/wordpress-kubernetes) vimos cómo ejecutar Wordpress (Wordpress + MySQL) en Kubernetes, del modo más básico, solo con dos Pods separados.
En este post vamos a poner la persistencia en elementos externos (volúmenes) para que los datos tengan un ciclo de vida separado de los Pods

## Mejorando la persistencia

Evidentemente la persistencia se hace dentro de los containers y esto no es ideal.
Vamos a agregar algo de storage.
Primero creamos el Persisten volumen, como yo estoy en Docker en Windows será del tipo **local-storage**
En este caso este tipo no permite Dynamic Provisioning, así que no hace falta StorageClass
Entonces creamos dos PV

``` yaml
kind: PersistentVolume
metadata:
  name: wp-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /run/desktop/mnt/host/c/k8svolume/wp #windows path
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - docker-desktop
```

``` yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  local:
    path: /run/desktop/mnt/host/c/k8svolume/mysql
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - docker-desktop
```          

Creamos un persisten volume claim para cada deployment

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
```

``` yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mywordpress-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 1Gi
```

Y por último modificamos los deployments de Wordpress y MySQL para declarar y montar el los claims

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-db
  labels:
    app: my-db
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-db
  template:
    metadata:
      labels:
        app: my-db
    spec:
      containers:
      - name: my-db
        image: mysql:5.7
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /var/lib/mysql
          name: my-db-pv
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "my-secret-pw"
        - name: MYSQL_DATABASE
          value: "my-db"
        - name: MYSQL_USER
          value: "my-user"
        - name: MYSQL_PASSWORD
          value: "my-secret-pw"
      volumes:
      - name: my-db-pv
        persistentVolumeClaim:
          claimName: mysql-pvc
```
``` yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-wordpress
  labels:
    app: my-wordpress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-wordpress
  template:
    metadata:
      labels:
        app: my-wordpress
    spec:
      containers:
      - name: my-wordpress
        image: wordpress:latest
        volumeMounts:
        - name: my-wp-pv
          mountPath: /var/www/html
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_PASSWORD
          value: "my-secret-pw"
        - name: WORDPRESS_DB_USER
          value: "my-user"
        - name: WORDPRESS_DB_NAME
          value: "my-db"
        - name: WORDPRESS_DB_HOST
          value: "mysql"
      volumes:
      - name: my-wp-pv
        persistentVolumeClaim:
          claimName: mywordpress-pvc
```
Y ya está, en el siguiente post mejoraremos la configuración.
Nos leemos en la próxima