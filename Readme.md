# Staging Test

This describes the testing of faulted and degraded volumes using mayastor on the Civo Staging platform.

When running workload from within kubevirt VMs, we are seeing volumes go into a faulted and degraded state:

```
 ID                                    REPLICAS  TARGET-NODE  ACCESSIBILITY  STATUS   SIZE
 2a417997-0fe0-4f72-9cff-57b4917edd8b  2         compute-02   nvmf           Faulted  16106127360
 7626258b-2477-48cd-b895-56bc5f741944  2         compute-01   nvmf           Faulted  16106127360
 48619dc7-97ea-4a48-a944-81d246f0af26  2         compute-03   nvmf           Faulted  16106127360
 9683d033-51df-4c06-9ad2-b5154f05a958  2         compute-02   nvmf           Faulted  16106127360
 3da078c0-3f9b-4365-8e58-c13c162b8911  2         compute-01   nvmf           Faulted  16106127360
 fc0949cc-3d9a-4d57-b7a3-63b05b119dbb  2         compute-01   nvmf           Faulted  16106127360
 bedfed84-d456-412b-8471-09920ddbc4db  2         compute-03   nvmf           Faulted  16106127360
 2a458b84-4844-4731-ba09-210c7a455482  2         compute-01   nvmf           Faulted  16106127360
 6823ddb5-1442-493a-b161-fa240db05f04  2         compute-03   nvmf           Faulted  16106127360
 89ed450a-8216-4a26-97a3-088506a6bb1e  2         compute-02   nvmf           Faulted  16106127360
 9fbf013b-01ed-40de-a8f8-adf137a46800  2         compute-03   nvmf           Faulted  16106127360
```



# General Monitoring 

Watch Mayastor Data Plane logs

```
stern -l app=mayastor -n mayastor
```

Watch Mayastor Volumes

```
watch -n 5 "./kubectl-mayastor get volumes && echo && ./kubectl-mayastor get volumes |wc -l"
```





# 01 - FIO Supercluster



Uses https://github.com/yasker/kbench

**Install**

```
cd 01-fio-supercluster
helm install fio-test .

```

**Uninstall**

```
helm delete fio-test
```

**Results**

No visible impact and the error could not be replicated. 

# 02 - CivoK3s Clsuter

**Install**

```
cd 02-cluster
kubectl apply -f civofirewall.yaml
kubectl apply -f cluster.yaml
kc get cm cluster-civo-nyc-01-5d96-33a6c0 -o json | jq -r .data.kubeconfig | d64 > kubeconfig

```

> WAIT FOR ALL TENANT NODES TO BE 'Reay'

```
KUBECONFIG=./kubeconfig kgno | wc -l
```



**Running the test**

```
KUBECONFIG=./kubeconfig kc apply -f tenant-daemonset.yaml
KUBECONFIG=./kubeconfig kgpw -A
```

**Uninstall**

```
kubectl delete -f cluster.yaml
kgpa -o wide | grep Termin | grep virt-laun | grep "0/1" | tr -s ' ' | cut -f 1,2 -d ' ' | while read line; do echo kubectl delete po --force -n $line; done | bash
```

**Results**

This triggered the issue with a 10 node clusters. A XXX node cluster worked without an issue.