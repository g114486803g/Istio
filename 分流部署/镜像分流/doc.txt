1.比如v1已经在运行了
test-v1.yaml
[root@VM_8_34_centos test]# kubectl get pods -n opx
NAME                       READY   STATUS    RESTARTS   AGE
http-v1-7656fb7df6-mh6h5   2/2     Running   0          105m

再创建一个共同的资源文件svc-all.yaml
[root@VM_8_34_centos test]# kubectl get svc -n opx
NAME       TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
http-all   NodePort   10.111.179.214   <none>        80:30099/TCP   124m
http-v1    NodePort   10.111.97.35     <none>        80:30090/TCP   126m
http-v2    NodePort   10.96.37.92      <none>        80:30092/TCP   17m



2.在运行
virt+rule.yaml 
[root@VM_8_34_centos test]# kubectl apply -f virt+rule.yaml
virtualservice.networking.istio.io/test-vir created
destinationrule.networking.istio.io/test-dr created

-----------继续将流量打到V1---------------
    - destination:
        host: http-all
        subset: v1
      weight: 100

--------------------------------------------------



3. 在运行v2 版本，并且确保v2版本运行正常
test-v2.yaml

[root@VM_8_34_centos test]# kubectl get pods -n opx
NAME                       READY   STATUS    RESTARTS   AGE
http-v1-7656fb7df6-mh6h5   2/2     Running   0          114m
http-v2-6d97475f88-cwgvj   2/2     Running   0          5m46s
---------------------------------------------
为了保险将部分20%的流量打到v2

   - destination:
        host: http-all
        subset: v1
      weight: 80
    - destination:
        host: http-all
        subset: v2
      weight: 20

5.再重置生效
[root@VM_8_34_centos test]# kubectl apply -f virt+rule.yaml
virtualservice.networking.istio.io/test-vir configured
destinationrule.networking.istio.io/test-dr unchanged

----------------------------------------------
生产一线环境推荐

    - destination:
        host: http-all
        subset: v2   <<<<<<-----------直接将v1改成v2
      weight: 100



------------------------------------------------------


[root@VM_8_34_centos test]# kubectl get -n opx virtualservices.networking.istio.io
NAME       GATEWAYS   HOSTS        AGE
test-vir              [http-all]   23m

[root@VM_8_34_centos test]# kubectl get -n opx destinationrules.networking.istio.io
NAME      HOST       AGE
test-dr   http-all   22m

测试
while true; do curl http-all; sleep 1; done

