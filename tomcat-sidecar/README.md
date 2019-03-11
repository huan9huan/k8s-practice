# use k8s to deploy tomcat war

## debug

```
kubectl exec tomcat-deployment-6c4f66cc69-cdlnq -- ls /root/apache-tomcat-7.0.42-v2/webapps
kubectl exec tomcat-deployment-78b695fc-hj6zr -- ls /root/apache-tomcat-7.0.42-v2/webapps
kubectl exec tomcat-deployment-5dd4b85fbf-6gxfl -- caat /root/apache-tomcat-7.0.42-v2/conf/server.xml
```