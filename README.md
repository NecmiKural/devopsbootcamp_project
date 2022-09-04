# 1) GCP'de Kubernetes cluster kurulumu

GCP'ye üye olup proje oluşturulduktan sonra sol üstteki menüden Kubernetes Engine kısmına girilir ve Kubernetes Engine apisi kullanılmak üzere etkinleştirilmelidir. 

Bu noktadan sonra cluster oluşturmak için arayüz veya cloud shell'den komutlar kullanılabilir:

#### 1) Cloud Shell ile Cluster Oluşturma

Ekranın sağ üst tarafından cloud shell açılır. ![](C:\Users\berat\AppData\Roaming\marktext\images\2022-09-04-15-08-36-image.png)

Ardından aşağıdaki gibi komut girilir ve birkaç dakika sonra clusterımız hazır olur.

```
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

## 3) Terraform ile Cluster Oluşturma



# 2) Kubernetes Üzerine MySQL Kurulumu




