1. slogin 포드 접속하기
먼저 명령어를 입력할 수 있는 관문인 slogin 포드의 이름을 확인하고 접속합니다.
```
kubectl get pods -n slurm
```
[결과]
```
NAME                             READY   STATUS             RESTARTS         AGE
slurm-controller-0               3/3     Running            0                45m
slurm-restapi-5468d6d478-xmxdk   1/1     Running            0                45m
slurm-worker-slinky-0            1/2     CrashLoopBackOff   15 (4m35s ago)   45m
```

```
kubectl exec -it <slogin-pod-name> -n slurm -- /bin/bash
```
