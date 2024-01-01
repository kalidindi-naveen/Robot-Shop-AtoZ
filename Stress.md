### Stress
```
It will exhaust all resources like CPU, Memory
```
```
FROM  almalinux:8
RUN yum update -y
RUN yum install stress-ng -y
CMD ["stress-ng","--help"]
```
```
docker build -t naveen2809/stress-ng:v1 .
```
```
docker push naveen2809/stress-ng:v1
```