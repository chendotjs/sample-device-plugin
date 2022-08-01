# sample-device-plugin

## 编译二进制

```
make go.build
```

## 编译镜像

```
make image
```

## 部署

```
# kubectl label nodes xiabingyao-lc0 sample-device=enable
# kubectl apply -f build/deploy/manifest/sample-device-plugin.yaml
```

## 查看

```
# kubectl describe nodes master-01
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                1090m (2%)  16 (33%)
  memory             560Mi (0%)  8532Mi (8%)
  ephemeral-storage  0 (0%)      0 (0%)
  hugepages-1Gi      0 (0%)      0 (0%)
  hugepages-2Mi      0 (0%)      0 (0%)
  sample.com/sample  0           0
```
出现sample.com/sample资源

ListAndWatch 上报资源相关的 kubelet 日志：
```
Aug 01 17:32:39 cce-alqjuwli-w6y7xfxs kubelet[6059]: I0801 17:32:39.156561    6059 endpoint.go:112] State pushed for device plugin sample.com/sample
Aug 01 17:32:47 cce-alqjuwli-w6y7xfxs kubelet[6059]: I0801 17:32:47.945322    6059 setters.go:343] Update capacity for sample.com/sample to 5
```

从 dp 汇报至 kubelet 到 update apiserver 间隔时间与 `--node-status-update-frequency` 参数有关，默认值是 10s.

## 测试
```
# cat  build/deploy/sample/pod-use-sample-resource.yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-pod
  labels:
    name: sample-pod
spec:
  containers:
    - name: sample-pod
      image: nginx:1.14.2
      resources:
        requests:
          memory: "50Mi"
          cpu: "50m"
          sample.com/sample: 1
        limits:
          memory: "150Mi"
          cpu: "500m"
          sample.com/sample: 1
      ports:
        - containerPort: 80

# kubectl apply -f build/deploy/sample/pod-use-sample-resource.yaml
```