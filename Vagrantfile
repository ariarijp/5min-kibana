# -*- mode: ruby -*-
# vi: set ft=ruby :

$td_agent_conf = <<CONFIG
<source>
  type tail
  format apache
  path /var/log/apache2/access.log
  pos_file /var/log/td-agent/kibana-apache-access.pos
  tag kibana.apache.access
</source>

<match kibana.apache.access>
  type elasticsearch
  include_tag_key true
  tag_key @log_name
  host localhost
  port 9200
  logstash_format true
  flush_interval 5s
</match>
CONFIG

$script = <<SCRIPT
wget -bqc https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.3.2.deb
wget -bqc https://download.elasticsearch.org/kibana/kibana/kibana-3.1.0.tar.gz
echo "Asia/Tokyo" > /etc/timezone
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
sed -i s/archive.ubuntu.com/ftp.jaist.ac.jp/ /etc/apt/sources.list
wget -nv http://packages.treasure-data.com/debian/RPM-GPG-KEY-td-agent
apt-key add ./RPM-GPG-KEY-td-agent
echo "deb http://packages.treasure-data.com/precise/ precise contrib" > /etc/apt/sources.list.d/treasure-data.list
apt-get update
apt-get install -y --force-yes td-agent apache2 openjdk-7-jre-headless libcurl4-openssl-dev make language-pack-ja
chmod 755 /var/log/apache2 -R
while true; do echo "156a38c5a829e5002ae8147c6cac20effe6cd065  elasticsearch-1.3.2.deb" | sha1sum -c - && break; sleep 10; done
dpkg -i elasticsearch-1.3.2.deb
update-rc.d elasticsearch defaults 95 10
/usr/share/elasticsearch/bin/plugin -s -i elasticsearch/marvel/latest
while true; do echo "effc20c83c0cb8d5e844d2634bd1854a1858bc43  kibana-3.1.0.tar.gz" | sha1sum -c - && break; sleep 10; done
tar xzf kibana-3.1.0.tar.gz
mv kibana-3.1.0 /var/www/html/kibana
/usr/lib/fluent/ruby/bin/fluent-gem install --no-ri --no-rdoc fluent-plugin-elasticsearch
echo "#{$td_agent_conf}" >> /etc/td-agent/td-agent.conf
service elasticsearch start
service td-agent restart
SCRIPT

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.hostname = "5min-kibana"
  config.vm.box = "ubuntu/trusty32"

  config.vm.provision "shell", inline: $script

  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 9200, host: 9200

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end
end
