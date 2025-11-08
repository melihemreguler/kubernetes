# kubernetes

## kubctl

```bash
kubectl apply -f microservices.ymal
```

- Biz `kubectl` ’ den microservices yaratmasını istediğimizde o bunu alıp api server’ a götüren garson gibidir.
    - Götürmeden önce `~/.kube/config` dosyalarını okur.
    - Kimlik bilgilerini alır.
    - Verdiğin yaml dosyasını json’ a çevirir çünkü biraz sonra http request ile **api server**’ a post edecektir.

## Api Server

- Bekçi görevi görür.
- Authentication, authorization yapar.
- RBAC işlerini yapar.
- Yaml dosyasının formatını kontrol eder.

## etcd

- Kuberntest’ in beyin hücresidir.
- key/value store’ dur.
- **Raft Consensus** algoritmasını kullanır. (verinin stabil olması için)
- Kubernetes’ in **state**’ inin tutulduğu yerdir.

<aside>
ℹ️

Mülakatlarda ayırt edici sorular olarak gelir.

</aside>

## Control Manager

- Her zaman **etcd**’ yi dinler.
- Yeni bir şey geldiği zaman aksiyon almaya çalışır.
- Yeni bir deployment var mı diye kontrol eder.
- Yeni bir deployment varsa **Deployment Controller**’ i devreye sokar.

<aside>
ℹ️

Bir sürü controller vardır.

</aside>

- Deployment Controller isteği açar bakar;
    - Bu serviste kaç ane replica istenmiş?
        - Replica stoğunu oluşturur.
    - Replica Set’ i oluşturduktan sonra **Deployment Controller**’ in asıl amacı olan
        - Running state’ ine ulaşılabiliyor mu kontrol eder.

## Kubernetes Scheduler

- Kubernetes’ in emlakçısıdır.
    - Pod’ lara en uygun olan node’ u bulur.
    - Bu pod için harika bir yer buldum diye cevap döner.
- İlk önce **unscheduled** olan pod’ ları bulur.
    - Daha sonra içindeki cpu ve memory isteklerine bakar, bu isteğe uygun hangi node’ lar var diye tarar.
- Bu pod’ un içerisinde başka istekler de olabilir;
    - Ben şu şu şu etiketlere sahip node’ lara gitmek istiyorum.
    - Şu etiketlere sahip nodun içinde durmak istemiyorum.

    <aside>
    ℹ️

    Bunlara genel olarak Node Affinity denir

    </aside>

- Sonuçta uygun olan node’ a assign eder.

## Kubelet

- Ortamdaki asıl yöneticidir.
- Bu kubelet’ in kubernetes’ in bütün node’ larında görebilirsin.
- Scheduler bana bir iş verdi hemen yapıyorum der.

## Container Runtime

- Docker container’ da işi fiziksel kısmını yapan eleman;
    - İmage’ ı indir → container’ i başlat → port’ ları aç → network ayarlarını yap → kullanıcı tanımladıysa gerekli volume’ ları, diskleri yarat. İlgili pod’ a mount olmasını sağla.
- Sonuç olarak container’ i ayağa kaldırır.