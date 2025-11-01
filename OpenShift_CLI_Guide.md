# 🧭 OpenShift Administration CMD Guide
> OpenShift CLI(`oc`) 명령어 정리 문서  
> 작성자: 사용자  
> 버전: 2025-10

---

## 📑 목차
1. [기본 로그인 및 버전 확인](#기본-로그인-및-버전-확인)
2. [프로젝트 관리](#프로젝트-관리)
3. [리소스 조회 및 설명](#리소스-조회-및-설명)
4. [리소스 생성 (App, Template, Deployment 등)](#리소스-생성-app-template-deployment-등)
5. [리소스 수정, 패치, 삭제](#리소스-수정-패치-삭제)
6. [Pod 디버깅 및 접근](#pod-디버깅-및-접근)
7. [로그, 이벤트, 노드 모니터링](#로그-이벤트-노드-모니터링)
8. [ConfigMap 및 Secret 관리](#configmap-및-secret-관리)
9. [볼륨 및 PVC 관리](#볼륨-및-pvc-관리)
10. [Probe, Autoscale, Rollout 관리](#probe-autoscale-rollout-관리)
11. [Service, Route, Ingress 관리](#service-route-ingress-관리)
12. [ImageStream / ImageStreamTag 관리](#imagestream--imagestreamtag-관리)
13. [Skopeo / 이미지 작업](#skopeo--이미지-작업)
14. [crictl / nsenter 명령어](#crictl--nsenter-명령어)

---

## 🧩 기본 로그인 및 버전 확인
```bash
oc login -u developer -p developer
oc login -u developer -p developer https://console-openshift-console.apps.ocp4.example.com
oc version
```

---

## 🧱 프로젝트 관리
```bash
oc new-project myproject
oc project myproject
oc delete project myproject
```

---

## 🔍 리소스 조회 및 설명
```bash
oc get pods -o wide
oc get pods --show-labels
oc get nodes
oc get operators
oc get clusteroperators
oc get ingress
oc get routes
```

### API 리소스 조회
```bash
oc api-resources
oc api-resources --namespaced
oc api-versions
```

### 리소스 설명
```bash
oc explain pods
oc explain pods.spec.containers.image
oc describe pod mypod
oc describe node master01
oc describe project myproject
```

---

## ⚙️ 리소스 생성 (App, Template, Deployment 등)
```bash
oc new-app -f myapp.yaml
oc new-app --template mysql-persistent   --param MYSQL_USER=operator --param MYSQL_PASSWORD=password
```

### Deployment & Service 생성
```bash
oc create deploy myapp --image=nginx --port=80 --replicas=3
oc create service clusterip myapp --tcp=80:80
oc expose deploy myapp
```

### StatefulSet & Headless Service
```bash
oc create service clusterip nginx --cluster-ip=None --tcp=80:80
oc create statefulset nginx --image=nginx:latest --port=80 --replicas=3 --service=nginx
```

### CronJob / Job
```bash
oc create job myjob --image=redhat/ubi9 -- /bin/bash -c "date"
oc create cronjob mycronjob --image=redhat/ubi9 --schedule "*/1 * * * *" -- date
```

---

## 🧩 리소스 수정, 패치, 삭제
```bash
oc edit pod mypod
oc replace -f mydeploy.yaml
oc replace --force -f mydeploy.yaml
oc delete pod mypod
oc delete pod -l tier=frontend
oc patch deploy mydeploy -p '{"spec":{"replicas":3}}'
```

---

## 🐚 Pod 디버깅 및 접근
```bash
oc debug node/master01
oc run mypod --image=nginx --port=80
oc run -it mypod --image=nginx -- /bin/bash
oc exec -it mypod -- /bin/bash
oc rsh mypod
oc attach mypod -it  # 종료: CTRL + P + Q
oc port-forward mypod 8080:80
```

파일 복사 및 동기화:
```bash
oc cp db.sql mydb:/tmp/
oc cp mypod:/var/www/html/index.html /tmp/
oc rsync mypod:/var/www/ /tmp/webfiles
```

---

## 📋 로그, 이벤트, 노드 모니터링
```bash
oc logs mypod
oc logs mypod -c mycontainer
oc get events
oc get events -n openshift-kube-controller-manager
```

관리자 로그 확인:
```bash
oc adm top pods
oc adm top nodes
oc adm node-logs master01 -u crio
oc adm node-logs master01 -u kubelet
```

---

## 🔐 ConfigMap 및 Secret 관리
```bash
oc create cm mycm --from-literal key1=config1 --from-literal key2=config2
oc create secret generic mysecret --from-literal key1=secret1 --from-literal key2=secret2
oc create secret tls secret-tls --cert ./my.crt --key ./mykey
```

마운트 설정:
```bash
oc set volume deploy/mydeploy --add --type secret   --secret-name mysecret --mount-path /mysecret
oc set env deploy/mydeploy --from secret/mysecret --prefix MYSQL_
```

Secret 추출 및 업데이트:
```bash
oc extract secret/mysecret -n demo --to /tmp/demo --confirm
oc set data secret/mysecret -n demo --from-file /tmp/demo/mypassword
```

---

## 💾 볼륨 및 PVC 관리
```bash
oc set volumes deploy/db-pod --add --name lvm-storage --type pvc   --claim-mode rwo --claim-size 1Gi --mount-path /var/lib/mysql   --claim-class lvms-vg1 --claim-name db-pod-pvc

oc set volumes deploy/web-pod --add --name nfs-volume   --claim-name nfs-pvc --mount-path /var/www/html
```

---

## 🩺 Probe, Autoscale, Rollout 관리
```bash
oc set probe deploy/mydeploy --readiness --get-url http://:8080/healthz
oc autoscale deploy/mydeploy --min 2 --max 20 --cpu-percent 50
oc rollout pause deploy/mydeploy
oc rollout resume deploy/mydeploy
oc rollout history deploy/mydeploy
oc rollout undo deploy/mydeploy
```

---

## 🌐 Service, Route, Ingress 관리
```bash
oc expose deploy mydeploy --type=ClusterIP --port=80
oc expose svc mysvc --name myroute
oc create ingress myingress --rule "www.example.com/*=mysvc:8080"
oc scale deploy mydeploy --replicas=3
```

---

## 🖼️ ImageStream / ImageStreamTag 관리
```bash
oc create is myimagestream
oc create istag myimagestream:v1.0   --from-image registry.ocp4.example.com:8443/redhattraining/myimagestream:v1.0
oc set image-lookup myimagestream
oc set triggers deploy/mydeploy --from-image php-ssl:1 --containers php-ssl
oc tag --alias php-ssl:1-234 php-ssl:1
```

---

## 🧰 Skopeo / 이미지 작업
```bash
skopeo login quay.io
skopeo inspect docker://registry.access.redhat.com/ubi9/httpd-24
skopeo copy docker://quay.io/skopeo/stable:latest docker://registry.example.com/skopeo:latest
skopeo delete docker://registry.example.com/skopeo:latest
skopeo sync --src docker --dest docker registry.access.redhat.com/ubi8/httpd-24 registry.example.com/httpd-24
oc image info registry.access.redhat.com/ubi9/httpd-24:1-233
```

---

## 🧩 crictl / nsenter 명령어
```bash
crictl ps -a
crictl logs mycontainer
crictl inspect mycontainer
crictl exec -it mycontainer cat /etc/redhat-release
lsns -p $PID
nsenter
```

---

## ✅ 참고
- OpenShift CLI 공식 문서: [https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)
- oc 명령어 도움말: `oc --help`
