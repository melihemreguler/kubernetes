# kind

- Basit olsun diye local’ de kurup inceleyeceğiz.
- Kubernetes seçenekleri arasında;
  - Minikube
  - Docker’ in üzerinde gelen kubernetes servisi
    Biz araştırıp istediğimizi kullanabiliriz.
- Kind cluster’ ini kurup onun üzerinden kendimize bir kubernetes cluster’ i yaratacağız.
- [kind.sigs.k8s.io](https://www.notion.so/Kind-29fbf8c71b74805fa572eca6ab3f9e12?pvs=21) adresinden → quick start → `brew install kind`
- Bizim nasıl bir cluster’ a ihtiyacımız var?
  - 1 tane master’ i
  - 2 tane node’ u olan, workrunner’ i olan bir cluster yaratmak istiyorum
- `kind: apiVersion:` kubernetes’ te bir şey yaratıyorsak bu iki alan zorunludur.
- `kind.x-k8s.io/v1alpha4` api version’ u kind ile birlikte gelen custom bir source’ dur.
- Bunlardan sonra da node’ lar tanımlanıyor.
- `kind-config.yaml` dosyasında
  - bir control-plane (master) ve 2 tane worker (node) tanımlıyoruz.
- `kind create cluster --config kind-config.yaml --name demo-cluster`
- node' larda kendi özel image' ı olan `kindest/node`' u indiriyor.
- Daha sonradan node' ları ayarlıyorum diyor.
- Config' leri ayarlıyor.
- İlk önce contrrol-plane' i ayarlıyor çünkü ilk önce beyini ayağa kaldırmamız lazım daha sonra worker node' ları buraya bağlamamız gerekli.
- Sonra CNI' ı indiriyor (Network interface' i ayarlayan araç) pod' ların birbirleri ile konuşabilmesi için.
- StorageClass' ı indiriyor.
  - Biz mesela volume yaratıp disk istediğimizde;
  - Kubernetes' in bu işi birine delege etmesi lazım.
  - Mesela AWS' de EBS volume yaratır.
- En son "joining worker nodes" ile worker node' ları beyin olan contrrol-plane' e bağlıyor. Kurulumu tamamlıyor.

- herhangi bir şey yapmak istiyorsak ilk olarak şu komutu çalıştırabiliriz.
  - `kubectl cluster-info --context <cluster-name>`
- Sistem bileşenlerinin olduğu "kube-system" namespace' ine bakalım.

  - `kubectl get pods -n kube-system`

- Evimizi bir cluster olarak düşünürsek;

  - Her bir kat bir node' dur.
  - Odalarına da namespace diyebiliriz.
  - Odaların içindeki eşyalar ise pod' lardır.

- `kube-apiserver-demo-cluster-control-plane` → master node' unun pod' u.
- etcd' ye bir şey yazıldığı zaman onları dinleyen, yapmaya çalışan controller manager var.

  - Birazdan bir deployment yapacağız, orada 3 tane replica isteyeceğiz. "controller manager" bunu dinleyecek ve 3 tane pod' u ayağa kaldıracak. `kube-controller-manager-demo-cluster-control-plane`

- Deployment yapmak için içinde manifest tanımlı olan bir örnek dosya oluşturalım.
- ## deployment.yaml
- ### Etiketleme
- cluster' imizde bir sürü resource olabilir, onları mantıklı bir şekilde kümelemek ya da listelerken filtrelemek için kullanabiliriz.

### spec

- Bu deployment' in neye benzeyeceğini tanımlıyoruz.
- Garsona şunu şunu getir dediğimiz yer.

### replicas

- Deployment, replica set yaratır, replica set de pod' ları yaratır. (replicasının 3 olmasını istiyoruz.)
- Yani 3 tane pod' un ayağa kalkmasını istiyoruz.

```yaml
containers:
  - name: nginx
    image: nginx:1.21
    imagePullPolicy: IfNotPresent # Eğer bu image localde yoksa çek
    ports:
      - containerPort: 80 # localimizde olduğumuzdan sertifikayla uğraşmamak için 80 portunu açıyoruz.
    resources:
      requests:
        memory: "64Mi" # Pod' un memory ihtiyacı
        cpu: "250m" # Pod' un cpu ihtiyacı (millicore)
      limits:
        memory: "128Mi" # Pod' un maximum memory kullanımı
        cpu: "500m" # Pod' un maximum cpu kullanımı
```

- `kubectl create -f deployment.yaml` ile deployment' ımızı yaratıyoruz.
- `kubectl logs -f <pod-ismi>` ile pod' un loglarına bakabiliriz. (-f follow)

- loglara tek tek bakmak yerine hepsine birden bakmak istersek, kubernetes cluster' ımıza bir logging setup yapmamız lazım.
- Bu kubernetes cluster' ı şu anda kapalı kutu gibi,

  - dışarıya açmak için bir service yaratmamız lazım.

- `kubectl get deploy`ile deployment' ımızı görebiliriz.
- `kubectl get rs` ile replica set' imizi görebiliriz.

- nginx' ten replica set türemiş, bu replica set' ten farklı deployment' lar üretilmiş.

- `kubectl get pods -o wide` ile pod' larımızı daha detaylı görebiliriz.
  - ip adresleri gibi.
- service yaratalım.
- ## service.yaml

- Dışarıdan 80 portundan (port) gelecek istekleri alıp pod' lardaki 80 portuna (targetPort) yönlendirecek bir service yaratıyoruz.

  - Hangi target' taki 80' e gidecek bunu selector ile belirtiyoruz.
  - app: nginx etiketine sahip pod' lara istekleri yönlendirecek.

- `kubectl get pods -l 'app=nginx'` ile app etiketi nginx olan pod' ları listeleyebiliriz.

```bash
$ kubectl get pods -l 'app=nginx'
NAME                     READY   STATUS    RESTARTS   AGE
nginx-665c8876f8-gxdgj   1/1     Running   0          80s
nginx-665c8876f8-s4ml6   1/1     Running   0          80s
nginx-665c8876f8-w2jb9   1/1     Running   0          80s
```

- `kubectl apply -f service.yaml` ile service' imizi yaratıyoruz.

```bash
$ kubectl apply -f service.yaml
service/nginx created
```

- `kubectl get svc` ile service' lerimizi listeleyebiliriz.
- apply komutu create or update yapar.

- `kubectl get svc` ile service' lerimizi listeleyebiliriz. (svc = service)

```bash
$ kubectl get svc
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP      10.96.0.1      <none>        443/TCP        8m26s
nginx        LoadBalancer   10.96.187.12   <pending>     80:31122/TCP   23s
```

- nginx service' inin external-ip' i pending görünüyor çünkü kind ortamında LoadBalancer tipi service' ler desteklenmiyor.
  - Biz de service' in tipini LoadBalancer' dan NodePort' a dönüştüreceğiz.
- service' in tipi LoadBalancer' dan NodePort' a dönüştürelim.

  - `kubectl apply -f service.yaml`

- Bir daha `kubectl get svc` ile service' lerimizi listeleyelim.

```yaml
kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        26h
nginx        NodePort    10.96.187.12   <none>        80:31122/TCP   26h
```

- kubectl port-forward ile local makinemizin bir portunu pod' un portuna yönlendirebiliriz.
  - `kubectl port-forward svc/nginx 8080:80`
- Artık local makinemizin 8080 portuna gelen istekler nginx service' inin 80 portuna yönlendirilecek.
- Tarayıcıdan `http://localhost:8080` adresine gittiğimizde nginx karşılama sayfasını görebiliriz.


