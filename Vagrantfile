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
wget -nv http://packages.treasure-data.com/debian/RPM-GPG-KEY-td-agent
apt-key add ./RPM-GPG-KEY-td-agent
echo "Asia/Tokyo" > /etc/timezone
rm -f /etc/localtime
ln -s /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
sed -i s/us.archive.ubuntu.com/ftp.jaist.ac.jp/ /etc/apt/sources.list
echo "deb http://packages.treasure-data.com/precise/ precise contrib" > /etc/apt/sources.list.d/treasure-data.list
apt-get update
apt-get install -y --force-yes td-agent apache2 openjdk-7-jre-headless libcurl4-openssl-dev make
chmod 755 /var/log/apache2 -R
wget -nv https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.0.1.deb
dpkg -i elasticsearch-1.0.1.deb
/usr/share/elasticsearch/bin/plugin -i elasticsearch/marvel/latest
service elasticsearch start
wget -nv https://download.elasticsearch.org/kibana/kibana/kibana-3.0.0milestone5.tar.gz
tar xzf kibana-3.0.0milestone5.tar.gz
mv kibana-3.0.0milestone5 /var/www/kibana
/usr/lib/fluent/ruby/bin/fluent-gem install --no-ri --no-rdoc fluent-plugin-elasticsearch
echo "#{$td_agent_conf}" >> /etc/td-agent/td-agent.conf
service td-agent restart
SCRIPT

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.hostname = "5min-kibana"
  config.vm.box = "precise32"
  config.vm.box_url = "http://files.vagrantup.com/precise32.box"

  config.vm.provision "shell", inline: $script

  config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :forwarded_port, guest: 9200, host: 9200

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "512"]
  end
end
