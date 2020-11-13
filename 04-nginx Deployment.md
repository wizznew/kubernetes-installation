### Single Instance Microservice: nginx

> ##### Deploy nginx
> ```
> $ kubectl create deployment nginx --image=nginx
> deployment.apps/nginx created
>
> $ kubectl get deployment
> NAME    READY   UP-TO-DATE   AVAILABLE   AGE
> nginx   0/1     1            0           13s
> ```
>
>
> ##### Deploy NodePort service
> ```
> $ kubectl create service nodeport nginx --tcp=80:80
> service/nginx created
>
> $ kubectl get svc
> NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
> kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        143m
> nginx        NodePort    10.107.231.7   <none>        80:32253/TCP   15s
> ```
>
>
> ##### Check and Verify
> ```
> $ kubectl get all -o wide
> NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
> pod/nginx-6799fc88d8-4v56n   1/1     Running   0          35m   10.244.1.2   kubeworker1.kubelocal   <none>           <none>
> 
> NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE    SELECTOR
> service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        176m   <none>
> service/nginx        NodePort    10.107.231.7   <none>        80:32253/TCP   33m    app=nginx
> 
> NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
> deployment.apps/nginx   1/1     1            1           35m   nginx        nginx    app=nginx
> 
> NAME                               DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES   SELECTOR
> replicaset.apps/nginx-6799fc88d8   1         1         1       35m   nginx        nginx    app=nginx,pod-template-hash=6799fc88d8
> ```
>
>
> ##### Let's now delete pod and check what will happen
>```
>$ echo && echo "Pod information before delete" && echo && kubectl get pod -o wide && echo && echo && echo "Delete pod nginx-6799fc88d8-4v56n" && kubectl delete pod nginx-6799fc88d8-4v56n && sleep 10 &&echo && echo && echo "Pod information after delete" && echo && kubectl get pod -o wide && echo
>
> Pod information before delete
> 
> NAME                     READY   STATUS    RESTARTS   AGE   IP           NODE                    NOMINATED NODE   READINESS GATES
> nginx-6799fc88d8-4v56n   1/1     Running   0          41m   10.244.1.2   kubeworker1.kubelocal   <none>           <none>
> 
> 
> Delete pod nginx-6799fc88d8-4v56n
> pod "nginx-6799fc88d8-4v56n" deleted
> 
> 
> Pod information after delete
> NAME                     READY   STATUS              RESTARTS   AGE   IP       NODE                    NOMINATED NODE   READINESS GATES
> nginx-6799fc88d8-7brrr   0/1     ContainerCreating   0          20s   <none>   kubeworker3.kubelocal   <none>           <none>
>```
>
>
After a successful pod deletion, kubernetes will automatically create a new one which running in different host/machine.

Above behaviour triggered by the number of replicaset when we deploy the microservice.

You may refer to section [Check and Verify] above to find out replicaset configured for the microservice
