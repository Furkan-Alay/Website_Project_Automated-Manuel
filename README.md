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
 * git clone-b local https://github.com/Furkan-Alay/Website_Project_Automated-Manuel.git
 * cd vprofile-project
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
 Description=Tomcat
 After=network.target
 [Service]
 User=tomcat
 Group=tomcat
 WorkingDirectory=/usr/local/tomcat
 Environment=JAVA_HOME=/usr/lib/jvm/jre
 Environment=CATALINA_PID=/var/tomcat/%i/run/tomcat.pid
 Environment=CATALINA_HOME=/usr/local/tomcat
 Environment=CATALINE_BASE=/usr/local/tomcat
 ExecStart=/usr/local/tomcat/bin/catalina.sh run
 ExecStop=/usr/local/tomcat/bin/shutdown.sh
 RestartSec=10
 Restart=always
 [Install]
 WantedBy=multi-user.target
-Tomcat
