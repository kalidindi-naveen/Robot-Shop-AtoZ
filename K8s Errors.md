## Kubernetes Errors

### ErrImagePull 
```
Image Not Exists on Hub
- when we provide wrong image name or tag.
```

### CrashLoopBackOff 
```
your pod is starting, crashing, starting again, and then crashing again
reason: container is not able to run.

Ex: we not provided **commad: ["sleep","200"]** in yml so it starts failing.
- name: almalinux
  image: almalinux
```
https://komodor.com/learn/how-to-fix-errimagepull-and-imagepullbackoff/
### ImagePullBackoff
```
Kubernetes can't pull the image you have specified and therefore keeps crashing.
- Network issue
- Authentication fail
```

### OOMKilled
```
I've found 2 different reasons:
Pod memory limit exceeded: If the Container continues to consume memory beyond its limit, the Container is terminated.
Node out of memory: If the kubelet is unable to reclaim memory prior to a node experiencing system OOM, then kills the container.
```