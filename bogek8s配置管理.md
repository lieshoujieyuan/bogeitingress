# bogeitingress
k8s bogeit  ingress 
### 第7关 k8s架构师课程之配置管理

大家好，我是博哥爱运维，K8s是如何来进行服务配置管理的呢？这节课博哥带大家来攻克这关。



##### Configmap, secret

对于容器而言，如果我们想修改一个容器镜像里面的配置，可以在Dockerfile这一步，将修改好的配置复制到镜像里面再重新打包，对于不用变动配置的镜像而言，这样做属于硬编码当然也可以，但一旦我们的镜像服务需要修改配置，那么就需要重新重新打包非常麻烦，对于K8s而言，对于配置这么重要的一个环节，自然有它的解决方案，那就是configmap（通常普通配置使用）和secret（对于一些机密配置信息使用），在上面的部分章节里面，有提前涉及到这部分内容，但没有进行仔细的讲解，这里就对它们作下详细的实践。

我这里会准备一个deployment的yaml配置，用busybox来作为服务镜像，通过一个完整的yaml就可以快速带你们理解并能熟练在K8s上使用configmap和secret，如果一下子理解不了，后面可以保存这份yaml来作来生产配置参考也是没问题的，用多了自然就熟了，yaml配置如下：

> configmap-secret-example-simple.yaml

```yaml
---
# configmap
# kubectl create configmap localconfig-env --from-literal=log_level_test=TEST --from-literal=log_level_produce=PRODUCE
apiVersion: v1
kind: ConfigMap
metadata:
  name: localconfig-env
data:
  log_level_test: TEST
  log_level_produce: PRODUCE

---
# configmap
# kubectl create configmap localconfig-file --from-file=localconfig-test=localconfig-test.conf --from-file=localconfig-produce=localconfig-produce.conf
apiVersion: v1
kind: ConfigMap
metadata:
  name: localconfig-file
data:
  localconfig-produce: |
    TEST_RELEASE = False
    PORT = 80
    PROCESSES = 0
    MESSAGE = Produce
  localconfig-test: |
    TEST_RELEASE = True
    PORT = 8080
    PROCESSES = 1
    MESSAGE = Test

---
# secret
# kubectl create secret generic mysecret --from-literal=mysql-root-password='!QAZ2wsx' --from-literal=redis-root-password='!2Boge' --from-file=my_id_rsa=/root/.ssh/id_rsa --from-file=my_id_rsa_pub=/root/.ssh/id_rsa.pub
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
  namespace: default
type: Opaque
data:
  my_id_rsa: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUEVBdG1jMjc0ZnBCOE5rNHNkMGhVYktKcVVjMnZpQUEveTIycUpvME5sVmtOalJtMDNpCmZTbEl6QUtZOWNLWmhxazhWM2lEeW1WSjVUcHM2QXZxK1dtWDMrbTlwS05icEgvOE1tTHk1cG1EaXdoUDNHdzAKeldaeFBmQXZhU2VCcGlwMTBmTmV4TWJUVXdCNjUrU01MSHN4QVlKZnBvS2ZaVDN1TVZpVHhzbWFQNUp2bmpCVgpMdUpVZXpKak4rcTByQjl1Mng5NnI3d1dlcGJSeGx0TmxoUlBhblpxMnJVSEU2Z3BlbjAxYTR3RGhZZU0zZkhWCm1qMnRTYjJUQXlGRjBQdXViNTJZWFhsMXpqdnNVMGNUMCtCcDNTSFlVTkdBZjVab2UxaW83QkZwSHR1TFdWZksKbnowR1cwUDBxK3NTM1VLTHlqaXN5aEYvR0Z4MlA3cjBhYmVYMXdJREFQQUJBb0lCQUFMME01b2JqUG9CaDF1MQpsUCtYdHlSQkJlQ1lVZCs5MDFJcGpWRFNodTVqWjFYaTBzajZDWlNadGFDU0RhcDBqK2tJZ1g1cmNHZUdMb2FqCktFb054Ymc3OEI3SUpIQ3pSejIvSjkxRUh6SFVQdVkxQTBWRmZoaGx5VlRrdHFuSG5XWDJ5RTVlZ0lHV1VFMmcKeGtjbzNlZ1lDVUNXWWIwODVwejdyOG91NzZCcXFZY1oyQVJGZWx1VjZ1ODZyajIzeG1QcGtvaFBLQS9QdERxSApsdmN1cDVHRFkxQ2NNMHFwRXJXejN6NTE4UjYzN1BNQ3hJM2VDaXAwMGhMMDJZdFkva2tvYUhnSHdpRWNYRXowCjhGeHpxR1BUc2VSQnBwcWlTaXh1a1Bjb285b1BuRVhnN25pVGI2VitCMUxZVmdjejVsQlQ0WEZQR1dNYlB6aGMKN0FXNjBBRUNnWUVBMlJQeGpYYThNZkQ5ZnhnVDdha01yZGVpanBETGt3VVVySm5KNVAycWErR1VQSld1N1RhUAozR3RFc0FoSE5VVlJKbHFzYUFJM2hhTGZsSysrSVdQZ3dJaWh2TTVLcnVHdjNzYmFudEltZUpkZmNlTjBqdHR6CmRKc3FFMHorajZ0YkdsNHNDek0wUmJyUEdxN3gwa0lTTkFpWXg3cUIzcUlBVGRkWHArRjVVRmNDZ1lFQTF4dHMKdC9Hc2xBd3JHN1RER2paZTZyNXJDQlloTGlJREptSjBsWXNNdnV1d01WRlcrNjNjQkRvL1pFbVNIbFpMMFhNVwp6dlRUV2x1WHpVTm9HQytDSWk1dDAwaUZsL2tlL3dVcG5qZnhtZm1NRHhNUkN2YzJESlRSeDAzaldzd0U3WTRxCnBLQ0dENTZyQW1hMms2VGJvKzBiWWJZRzBLb0JKM3lyVS94WlJJRUNnWUVBeTh6Qm0wWllXU3EvVTRydmFyakQKUnBLajh1VE51d0dTSDFsaXlzREJ0dmJaajBlYWl1b210ZkdmVXdUeWxYaTJieVBCcVBQcnpETFZaV3A1UGkvZQoyZU5zdFMyWHdBZnliVnlUODNlbzFwNkc1UDErYUlCdkxKSmdNL1BNS21YZDZpdHZmalA0dWc1aFBpdnNIWjNhCktTL0kvL3FCNHRxRkhvK0ZvLzl6UFpFQ2dZQTVQSGhnUFBlZzg5Z3BhS1BnL3VXbWJ3WUh3ZlBVMWtLbVhjQVAKNlZGOEl6amk5M1pDU0ZUdDN4N3VMMUt2dG1JNW5mc3RIQ2FBdnk0WkdON0V5U2hHdHJxbFVlWDB1LzYwKzYzSApDYmJKTjQwYW1nV0kwS0h2R1ZENFRmZTgwOTczNTBYY1NVbEZNUEx0QWErSWZuRmpIbnBGdUcvNTY3V2c3K0tkCjIwVmRnUEtCZ0UrK1BZRUZKK2ZQeGtMeFRweUw5MDVHSVBBS00wM0MzV2J6b0lzSEZzbXZObWN6V2piK2hTWGsKTmVVZ0VNSnhjNytDN21LckkxZEcweFhVcVhqM1dvMjVxY0ZQUFYrMHZ1UmVnWk10aStpTW1zcTUyR29Xdld4egpVbmhpdnpyOXFsV0lzeHVzdTZjbzdxZ1gxeHl5L2d2SGxWYWdrR1hHZi9iSEtqZmVYeHdHCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
  my_id_rsa_pub: c3NoLXJzYSBUVFRUQjNOemFDMXljMkVUVFRURFRGVEJUVFRCVEZDMlp6YnZoK2tIdzJUaXgwNkZSc29tcFJ6YStJVEQvTGJhb21nNDJWV0YyTkdZN2VKOUtVak1UcGoxd3BtR3FUeFhlSVBLWlVubE9tem9DK3I1YVpmZjZiMmtvMXVrZi93eVl2TG1tWU9MQ0UvY2JUN05abkU5OEM5cEo0R21LblU1ODE3RXh0TlRUSHJuNUl3c2V6RUJnbCttZ3A5bFBlNHhXSlBHeVpvL2ttK2VNRlV1NGxSN01tTTM2clNzSDI3YkgzcXZ2Qlo2bHRIR1cwMldGRTlxZG1yYXRGY1RxQ2w2ZlRWcmpUT0ZoNHpkOGRXYVBhMUp2Wk1ESVVYRis2NXZuWmhkZVhYT08reEY1eE03NEduZElkaEYwWUIvbG1oN1dLanNFV2tlMjR0WlY4cWZQRlpZNC9TcjZ4TGRGb3ZLT0t6S0VYOFlYSFkvdXZScHQ1Zlggcm9vdEBub2RlLTEK
  mysql-root-password: IVFBWjJ3c3g=
  redis-root-password: ITJCb2dl

---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: test-busybox
  name: test-busybox
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      run: test-busybox
  template:
    metadata:
      labels:
        run: test-busybox
    spec:
      containers:
      - name: test-busybox
        image: busybox
        args:
          - /bin/sh
          - -c
          - >
              echo "-------------------------------------------------";
              echo "TEST_ENV is:$(TEST_ENV)";
              echo "-----------------------
