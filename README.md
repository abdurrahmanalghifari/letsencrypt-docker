# Automatic Regenerate LetsEncrypt dihost Docker (nginx)

[Let's Encrypt](https://letsencrypt.org) is a free, automated, and open certificate authority brought to you by the non-profit Internet Security Research Group (ISRG).

> Tutorial install letsencrypt ssl di host docker container.
> (dan nginx-nya  didalam docker container ).

Install di host.
``` sh
yum -y install epel-release certbot
mkdir -p /opt/stagging/nginx/conf/.well-known/acme-challenge
``` 
buat symlink agar bisa otomatis autorenew nantinya.
```sh
mkdir -p /opt/stagging/nginx/conf/ssl2
cd /etc/letsencrypt/
mv archive archive-bak
ln -s /opt/stagging/nginx/conf/ssl2 archive
```
### Config .well-known taruh di setiap vhost(subdomain)
nanti akan kita daftarkan SSL pada semua domain serta sub-domain tersebut.
karena LetsEncrypt harus add manual setiap subdomain baru. 
untuk *.domain.tld itu tidak mencakup keseluruhan.


    # Blok sebelum tutup paling bawah
    #---ACME WELLKNOWN---#
    location /.well-known {
          alias /opt/stagging/nginx/conf/.well-known;
    }

### Generate SSL 
lalu jalankan perintah ini untuk generate semua subdomain. silahkan listing kebutuhannya.

```sh
certbot certonly --rsa-key-size 4096 --webroot --agree-tos --no-eff-email --email age@domain.com -w /opt/stagging/nginx/conf -d domain.com -d www.domain.com -d cms.domain.com -d apis.domain.com
```

### Enable Konfigurasi
setelah selesai generate SSL nya. tambahkan konfig SSL ini di vhost masing-masing domain dan subdomain.
``` sh
#SSL LETSENCRYPT
ssl_certificate /opt/stagging/nginx/conf/ssl2/domain.com/fullchain1.pem;
ssl_certificate_key /opt/stagging/nginx/conf/ssl2/domain.com/privkey1.pem;
```

### Test Nginx & Reload
```sh
docker exec -it stagging /opt/nginx/sbin/nginx -t
docker exec -it stagging /opt/nginx/sbin/nginx -s reload
```
# Finish

#######################################


### Enabling Crontab reload nginx di HOST & Autorenew nya
```sh 
crontab -e
```

```sh
#NTPUPDATE
*/45 * * * * /usr/sbin/ntpdate -s -b -p 8 -u 0.id.pool.ntp.org
#SSL REGEN
#setiap jam 09:00 WIB 3 bulan sekali.
0 9 * */3 1 /usr/bin/certbot renew --quiet
#setiap jam 09:10 WIB 3 bulan sekali.
10 9 * */3 1 docker exec -it stagging /opt/nginx/sbin/nginx -s reload

```
