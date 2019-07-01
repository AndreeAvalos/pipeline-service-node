# Sitio Web
Este es un sitio web de prueba para Kubernetes más Rancher para implementar un Pipeline básico

### Pre-requisitos 📋
1. Conocimientos en Docker
2. Servidor rancher en cualquier plataforma. (LOCAL o NUBE)
3. Tiempo

### Instalación 🔧
1. Crear Archivo .rancher-pipeline.yml 
```
nano .rancher-pipeline.yml
```
```

stages:
- name: Crear Imagen
  steps:
  - publishImageConfig:
      dockerfilePath: ./Dockerfile
      buildContext: .
      tag: andreeavalos/pipeline-server
      pushRemote: true
      registry: index.docker.io
- name: Crear en k8s
  steps:
  - applyYamlConfig:
      path: ./deployment.yaml
timeout: 60
notification: {}
```
2. Crear Archivo deployment.yaml
```
nano deployment.yaml
```
```
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: test
  namespace: entrega-final
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      workload.user.cattle.io/workloadselector: deployment-entrega-final-test
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      labels:
        workload.user.cattle.io/workloadselector: deployment-entrega-final-test
    spec:
      containers:
      - image: czdev/miws
        imagePullPolicy: Always
        name: test
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: dockerhub
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
---      
apiVersion: v1
kind: Service
metadata:
  annotations:
    field.cattle.io/targetWorkloadIds: '["deployment:entrega-final:test"]'
    workload.cattle.io/targetWorkloadIdNoop: "true"
    workload.cattle.io/workloadPortBased: "true"
  labels:
    cattle.io/creator: norman
  name: nginx-loadbalancer
  namespace: entrega-final
spec:
  externalTrafficPolicy: Cluster
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    workload.user.cattle.io/workloadselector: deployment-entrega-final-test
  type: LoadBalancer 

```
3. Crear Dockerfile
```
FROM ubuntu:18.04
RUN apt-get update
RUN apt-get install -y nginx
WORKDIR /var/www/html
RUN rm *
COPY index.html .
#ADD carpeta /root/
EXPOSE 80
#ENV VAR_ENTORNO contenido
CMD /usr/sbin/nginx -g "daemon off;"
```
4. Crear pagina principal
```
<html>
<head>
<title>
Pagina principal
</title>
</head>
<body>
PAGINA PARA SEMINARIO DE SISTEMAS 1
HECHO EN RANCHER
CON ENTREGA CONTINUA
</body>
</html>
```
5. Cada vez que se hace un commit, rancher lo toma automaticamente

* [Sergio Mendez](https://www.youtube.com/watch?v=k4y776PqTwI)-Guia de instalacion de rancher