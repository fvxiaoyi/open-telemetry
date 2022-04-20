Build jar to docker image. It as pod init container, mount jar volume share to pod.

```text

docker build -t ebinsu/agent:1 .

```

eg:

```text

apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order-service
      component: java
  template:
    metadata:
      labels:
        app: order-service
        component: java
    spec:
      initContainers:
      - name: otel-javaagent
        image: ebinsu/agent:1
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: otel-javaagent
          mountPath: /otel-javaagent
      containers:
      - image: ebinsu/order-service:27
        imagePullPolicy: IfNotPresent 
        name: order-service
        env:
        - name: JAVA_OPTS
          value: "-XX:+UseG1GC -XX:MaxRAMPercentage=75 -XX:MaxMetaspaceSize=128m -Xss256k -XX:G1PeriodicGCInterval=900k -javaagent:/otel-javaagent/opentelemetry-javaagent.jar"
        - name: OTEL_EXPORTER_OTLP_ENDPOINT
          value: "http://opentelemetry-collector:4317"
        - name: OTEL_RESOURCE_ATTRIBUTES
          value: "service.name=order-service,service.namespace=defalut"
        - name: OTEL_METRICS_EXPORTER
          value: "prometheus"
        ports:
        - name: http 
          containerPort: 8080
        - name: metrics
          containerPort: 9464
        volumeMounts:
        - name: otel-javaagent
          mountPath: /otel-javaagent
      volumes:
      - emptyDir: {}
        name: otel-javaagent


```