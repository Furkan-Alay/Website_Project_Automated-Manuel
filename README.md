# Manuel kurulum için aşağıdaki uyarılara ve komutlara dikkat edin.
### Manuel Kurulum için ön gereklilikler
*  Oracle VM Virtualbox
* Vagrant
* Vagrant eklentileri
* Gitbash ve diğer terminal editörleri
- Vagrant eklentisini yüklemek için şu komutu çalıştırabilirsiniz.
* vagrant plugin install vagrant-hostmanager
 Sanal Makine kurulumu için şunları yapın
 1. Kaynak kodu klon edin
 2. Repo içerisine girin
 3. cd into vagrant/Manual_provisioning kısmına girin
 
 -Sanal makinenizi çalıştırın
 * vagrant up
 -Sanal Makinenizi çalıştırdıktan sonra provizyonlanacak olan servislerimiz bunlardır.
 1. Nginx
 2. Tomcat
 3. RabbitMQ
 4. Memcache
 5. ElasticSearch
 6. MySQL
 
 ## MYSQL Kurulumu
 - db adındaki Sanal makineye giriş yaptık.
 * vagrant ssh db01
 - İşletim sistemimizdeki paketleri güncelledik
 * dnf update-y
 - Repository oluşturduk
 * dnf install epel-release-y
 - MariaDB paketini kurduk
 * dnf install git mariadb-server-y
 - mariadb servisimizi açık ve çalışır halde bıraktık.
 * systemctl start mariadb
 * systemctl enable mariadb
 -Root kullanıcısı için bir şifre oluşturmalıyız.
 * mysqladmin -u root password 'NEWPASSWORD'
 -Database root kullanıcısı oluşturalım
 * mysql-u root-padmin123
 -Gerekli yetkileri bu kullanıcıya verelim
 * mysql> create database accounts;
 * mysql> grant all privileges on accounts.* TO 'admin'@'localhost' identified by 'admin123';
 * mysql> grant all privileges on accounts.* TO 'admin'@'%' identified by 'admin123';
 * mysql> FLUSH PRIVILEGES;
 * mysql> exit;
