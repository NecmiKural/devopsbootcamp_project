# GCP'da Kubernetes cluster kurulumu

GCP'ye üye olup proje oluşturulduktan sonra sol üstteki menüden Kubernetes Engine kısmına girilir ve Kubernetes Engine apisi kullanılmak üzere etkinleştirilmelidir. 

Bu noktadan sonra cluster oluşturmak için arayüz veya cloud shell'den komutlar kullanılabilir:

### 1) Cloud Shell ile Cluster Oluşturma

Ekranın sağ üst tarafından cloud shell açılır. ![](C:\Users\berat\AppData\Roaming\marktext\images\2022-09-04-15-08-36-image.png)

Ardından aşağıdaki gibi komut girilir ve birkaç dakika sonra clusterımız hazır olur.

```shell
gcloud container --project "devopsbootcamp-359519" clusters create-auto "devopscluster" --region "europe-west3" --release-channel "regular" --network "projects/devopsbootcamp-359519/global/networks/default" --subnetwork "projects/devopsbootcamp-359519/regions/europe-west3/subnetworks/default" --cluster-ipv4-cidr "/17" --services-ipv4-cidr "/22"
```

Buradaki alanlar kendi oluşturmak istediğiniz projeye göre doldurulmalıdır:

- --project "çalıştığınız projenin idsi"

- clusters create-auto "oluşturacağınız clusterın adı"

- --region "projenizin bulunacağı region"

- --network "projects/proje id’si/global/networks/default"

- --subnetwork "projects/proje id’si/regions/projenizin bulunacağı region/subnetworks/default"

### 2) GCP Arayüzünden Cluster Oluşturma

Sol üstteki panelden Kubernetes Engine'ne girilir ve Create Cluster denilip Autopilot veya Manual cluster seçilir. Cluster adı ve regionu girilip ekstradan istenilen ayar varsa yapıldıktan sonra cluster hazır olur. 

![](C:\Users\berat\AppData\Roaming\marktext\images\2022-09-04-15-28-13-image.png)

### 3) Terraform ile Cluster Oluşturma



# Kubernetes Üzerine MySQL ve WordPress Kurulumu

Oluşturulan clustera girmek için cloud shell'e şu komut yazılır:

```shell
gcloud container clusters get-credentials devopsboot --region europe-west3 --project devopsbootcamp-359519
```

- Proje, credentials ve region kısmına kendi bilgilerinizi yazmalısınız. 

Böylece yapılacak kubernetes işlemleri bu cluster ile olacaktır.

MySQL ve WordPress verileri saklayabilmek için PersistentVolumes(PV) ve PersistentVolumeClaims(PVC) kullanır. PV, cluster içerisinde yönetici tarafından ayrılan minik bi depolama alanıdır, kubernetes tarafından da dinamik olarak kullanılmak üzere ayrılabilir. PVC ise, kullanıcı tarafından PV kullanabilmek için alan talep edilmesidir. PV ve PVC podun yaşam döngüsünden bağımsız çalışır; yeniden başlamalara, silmelere karşı dirençli olur.

Öncelikle kendimize çalışma dosyası ayıralım:

```shell
mkdir wordpress-mysql && cd wordpress-mysql
```

Kullanacağımız dosyaları çalıştıracağımız *setup.yaml* isminde tek bir dosya oluşturabiliriz. Bu dosyanın içerisinde gizli kalacak şifremizi, MySQL ve WordPress için ayarları barındırabiliriz. Bunun için şu komutu gireriz:

```shell
cat <<EOF >./setup.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=sifreniz
EOF
```

- burada cat komutu yerine vim setup.yaml yazarak da içini doldurabiliriz.

Burada şifremizi tutacak yeri hazırlamış oluyoruz. sifreniz yazan kısma database için kullanılacak şifre girilir.

Şimdi MySQL ayarlama dosyasını hazırlayabiliriz. Bunun için de öncelikle vim mysql-deployment.yaml ile dosyamız oluşturulur ve içine girilip doldurulur:

```shell
vim mysql-deployment.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  ports:
    - port: 3306
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

- kind: ne işe yarayacağı, hangi tür olduğunu belirtir

- accessModes: nasıl çalışacağını belirtir

- storage: ne kadar alan ayrılacağını belirtir

- selector altında matchLabels: eşleşeceği uygulamayı belirtir

- containers altında image: hangi imajı kullanacağını belirtir

- MYSQL_ROOT_PASSWORD: database için kullanacağımız şifreyi istiyoruz burada. valueFrom olarak oluşturduğumuz secretKey dosyasını referans gösterip mysql-pass tanımlamasını password olarak kullanacağını belirtiyoruz.

- volumeMounts kısmında persistent storage'ı nerede kullanacağını belirtiyoruz

Aynı şekilde WordPress ayar dosyamız hazırlanır:

```shell
vim wordpress-deployment.yaml
```

```yaml
apiVersion: v1
kind: Service
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:4.8-apache
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          value: wordpress-mysql
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-pass
              key: password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

- WordPress containerı web sitesi verileri için PersistentVolume'e /var/www/html konumunda bağlanır

- WordPress için gerekli olan database bilgilerini bir önceki dosyada isimlendirmiştik. Onları WORDPRESS_DB_HOST için çağırıyoruz böylece WordPress database'e Service ile bağlanabiliyor. Şifreyi de böylece alabiliyoruz. 

Şimdi hazırladığımı bu iki ayar dosyamızı kurulum dosyamıza bağlayabiliriz.

```shell
cat <<EOF >>./setup.yaml
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml
EOF
```

Böylece `setup.yaml` WordPress ve MySQL kurulumu için gerekli bilgilere sahip oluyor. Şimdi bunları yedirebiliriz

```shell
kubectl apply -k ./
```

Çalışıyor mu diye kontrol edebiliriz:

- Secret çalışıyor mu diye bakmak için:

```shell
kubectl get secrets
```

- Çıktısı şöyle olmalı:

```shell

```
