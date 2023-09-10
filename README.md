# DockerRegistry
Docker Registry Kurulumu

## Daemon Dosyası Yapılandırması(SSL Kullanılmayacaksa)
/etc/docker/daemon.json dosyası oluştur ve aşağıdaki json'ı ekle
```
{
    "insecure-registries":[
        "localhost:5000"
    ]
}
```

Sırasıyla aşağıdakileri çalıştı
```
sudo systemctl daemon-reload
```
```
sudo systemctl restart docker
```

## Authentication Yapıladırması

Docker registry credential için dosya oluştur
```
mkdir -p Docker_registry/auth
```

Daha sonra, parolayla htpasswd korumalı bir kullanıcı oluşturmak için bir httpd kapsayıcısı çalıştır

```
cd Docker_registry
```

```
docker run --entrypoint htpasswd httpd:2 -Bbn <KULLANICI-ADI> <ŞİFRE> > auth/htpasswd
```
çalıştırdıktan sonra credentials bilgilerini auth/htpasswd dosyası içerisine yazacaktır.

## Docker Registry Kurulumu
Tüm işlemler yapıldıktan sonra registry kurulumu gerçekleştirilebilir.

Kurmak için aşağıdaki komutu çalıştırın

```
docker run -itd \
  -p 5000:5000 \
  --name registry \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry
```

## Login İşlemleri
Login olabilmek için aşağıdaki komutu çalıştırın

```
docker login  <IP-ADRES>:5000 -u <KULLANICI-ADI> -p <ŞİFRE>
```

## Push İşlemi
Bir docker image'ini private registry'ye pushlamak için image'e tag vermelisiniz.

Örneğin;

Docker Hub ile centos image'ini çekelim

```
docker pull centos
```

Daha sonra çektiğimiz image e tag verelim

```
docker tag centos:latest <IP-ADRES>:5000/centos
```
tag verme işlemini gerçekleştikten sonra artık image'imizi private registry'ye pushlayabiliriz.
Bunun için aşağıdaki komutu kullanalım

```
docker push <IP-ADRES>:5000/centos
```
komutu çalıştırdaktan sonra image'i göndermeye başlayacaktır.Yükleme bittiğinde image'inizi artık pull edebilirsiniz.

## Pull İşlemi

Pull işlemi için aşağıdaki komutu kullanabilirsiniz.

```
docker pull  <IP-ADRES>:5000/centos
```

NOT:Pull ve Push işlemleri için eğer credentials ayarlanmışsa mutlaka giriş yapılmalı
