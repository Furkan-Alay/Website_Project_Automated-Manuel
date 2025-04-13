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
 -Kaynak kodu indirip ve veri tabanını başlatalım.
 * cd /tmp/
 * git clone https://github.com/Furkan-Alay/Website_Project_Automated-Manuel.git
 * cd Website_Project_Automated-Manuel
 * mysql-u root-padmin123 accounts < src/main/resources/db_backup.sql
 * mysql-u root-padmin123 accounts
 -Tablo bilgisini görebiliriz
 * mysql> show tables;
 * mysql> exit;
 -Mariadb servisimizi yeniden çalıştıralım.
 * systemctl restart mariadb
 ## RabbitMQ Kurulumu
 -RabbitMQ içerisine girdik
 * vagrant ssh rmq01
 -Son güncellemeleri bu komutla alalım
 * dnf update-y
 -Epel Reposunu kuralım
 * dnf install epel-release-y
 -Gereklilikleri indirelim
 * sudo dnf install wget-y
 * dnf-y install centos-release-rabbitmq-38
 * dnf--enablerepo=centos-rabbitmq-38-y install rabbitmq-server
 * systemctl enable--now rabbitmq-server
 -Kullanıcı oluşturalım,test edelim ve admin yetkisi verelim.
 * sudo sh-c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
 * sudo rabbitmqctl add_user test test
 * sudo rabbitmqctl set_user_tags test administrator
 * rabbitmqctl set_permissions-p / test ".*" ".*" ".*"
 * sudo systemctl restart rabbitmq-server
 ##Tomcat Kurulumu
 -Tomcat Sanal Makineye girdik
 * vagrant ssh app01
 -Son güncellemeleri aldık
 * dnf update-y
 -Epel Reposunu kuralım
 * dnf install epel-release-y
 -Gereklilikleri kuralım
 * dnf-y install java-17-openjdk java-17-openjdk-devel
 * dnf install git wget-y
 -/tmp/ içerisine girdik
 * cd /tmp/
 -Tomcat indirdik ve sıkıştırdık.
 * wget https://archive.apache.org/dist/tomcat/tomcat-10/v10.1.26/bin/apache-tomcat-10.1.26.tar.gz
 * tar xzvf apache-tomcat-10.1.26.tar.gz
 -Tomcat kullanıcısnı oluşturduk
 * useradd--home-dir /usr/local/tomcat--shell /sbin/nologin tomcat
-Tomcat home klasörü içerisine verileri aktardık.
 * cp-r /tmp/apache-tomcat-10.1.26/* /usr/local/tomcat/
-Tomcat home klasörünün sahibini tomcat kullanıcısı yaptık.
 * chown-R tomcat.tomcat /usr/local/tomcat
-Tomcat kullanıcısı için service dosyasını oluşturalım ve aşağıdaki bilgileri bu dosyaya aktaralım
 * vi /etc/systemd/system/tomcat.service
 * [Unit]
 * Description=Tomcat
 * After=network.target
 * [Service]
 * User=tomcat
 * Group=tomcat
 * WorkingDirectory=/usr/local/tomcat
 * Environment=JAVA_HOME=/usr/lib/jvm/jre
 * Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
 * Environment=CATALINA_HOME=/usr/local/tomcat
 * Environment=CATALINE_BASE=/usr/local/tomcat
 * ExecStart=/usr/local/tomcat/bin/catalina.sh run
 * ExecStop=/usr/local/tomcat/bin/shutdown.sh
 * RestartSec=10
 * Restart=always
 * [Install]
 * WantedBy=multi-user.target
-Sistem dosyasını yeniden başlatalım
* systemctl daemon-reload
-Tomcat servisimizi başlatalım ve açık hale getirelim.
* systemctl start tomcat
* systemctl enable tomcat
##Artık uygulamamızı Build ve Deploy işleminden geçirebiliriz.
-Maven kuralım
 * cd /tmp/
 * wget
 * https://archive.apache.org/dist/maven/maven-3/3.9.9/binaries/apache-maven-3.9.9-bin.zip
 * unzip apache-maven-3.9.9-bin.zip
 * cp-r apache-maven-3.9.9 /usr/local/maven3.9
 * export MAVEN_OPTS="-Xmx512m"
-Kaynak kodu indirelim
* git clone https://github.com/Furkan-Alay/Website_Project_Automated-Manuel.git
-Konfigürasyon güncellemesi yapalım
 * cd Website_Project_Automated-Manuel
 * vim src/main/resources/application.properties
-Kodumuzu build edelim(Website_Project_Automated-Manuel bu dizinde olmalıyız)
* /usr/local/maven3.9/bin/mvn install
-Artifact Deploy edelim
 * systemctl stop tomcat
 * rm-rf /usr/local/tomcat/webapps/ROOT*
 * cp target/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
 * systemctl start tomcat
 * chown tomcat.tomcat /usr/local/tomcat/webapps-R
 * systemctl restart tomcat
 ##Son olarak NGINX Kurulumu yapalım
 -NGINX VM açtık ve root kullanıcısına geçtik
 * vagrant ssh web01
 * sudo-i
 -Son güncellemeleri aldık
 * apt update
 * apt upgrade
 -NGINX İndirdik
 * apt install nginx-y
 -NGINX konfigürasyon dosyası oluşturalım ve içerisinde bunları yazalım
 * vi /etc/nginx/sites-available/vproapp
 * upstream vproapp {
 * server app01:8080;
 * }
 * server {
 * listen 80;
 * location / {
 * proxy_pass http://vproapp;
 * }
 * }
 -Default olan NGINX Konfigürasyon dosyasını silelim
 * rm-rf /etc/nginx/sites-enabled/default
 -Websitemize giriş yapmak için kısayol oluşturalım.
 * ln-s /etc/nginx/sites-available/vproapp /etc/nginx/sites-enabled/vproapp
 -NGINX Yeniden çalıştırdık
 * systemctl restartnginx
###TÜM kurulumlar bittiğine göre app vm içerisindeki Statik IP bilgisini kullanarak Websitemize erişebiliriz.

# Otomatik Kurulum
* Otomasyon için oluşturduğum 2.klasöre girmelisiniz.
* Vagrantfile dosyamın olduğu yere gelmelisiniz.
* Vagrantfile dosyasını vagrant up komutuyla çalıştırmanız yeterli olacaktır.
