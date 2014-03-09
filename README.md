Storm Cluster for the Denali Cluster
===============

# Servers:
## Nimbus and UI
m2-denali.hwx.test

## Supervisors
u1-denali.hwx.test
u2-denali.hwx.test
u3-denali.hwx.test
u4-denali.hwx.test



## Storm Cluster Setup.
vagrant cluster
Notes: JVM installed.  JAVA_HOME set.  Java in the PATH

### Create Storm Account and Directories
pdsh -g denali/storm 'groupadd storm; useradd --gid storm --home-dir /home/storm --create-home --shell /bin/bash storm' | dshbak

4 nodes
### Get the distribution
wget http://apache.claz.org/incubator/storm/apache-storm-0.9.1-incubating/apache-storm-0.9.1-incubating.tar.gz

Need to get this artifact to the vagrant host directory so it can be picked up below.

### Deploy:
pdsh -g denali/storm 'cd /vagrant;tar -zxf apache-storm-0.9.1-incubating.tar.gz -C /usr/share; mv /usr/share/apache-storm-0.9.1-incubating /usr/share/storm-0.9.1;chown -R storm:storm /usr/share/storm-0.9.1;mkdir -p /etc/storm' | dshbak

pdsh -g denali/storm ' mkdir /etc/storm;cd /etc/storm; git clone https://github.com/dstreev/cluster_configs.git conf;cd conf;git checkout -b denali.storm --track origin/denali.storm; chown -R storm:storm /etc/storm/conf' | dshbak
pdsh -g denali/storm 'mkdir -p /var/log/storm;chown storm:storm /var/log/storm;rm /usr/share/storm-0.9.1/logback/cluster.xml;ln -s /etc/storm/conf/cluster.xml /usr/share/storm-0.9.1/logback/cluster.xml' | dshbak
pdsh -g denali/storm 'ln -s /usr/share/storm-0.9.1/bin/storm /usr/bin/storm;rm /usr/share/storm-0.9.1/conf/storm.yaml;ln -s /etc/storm/conf/storm.yaml /usr/share/storm-0.9.1/conf/storm.yaml' | dshbak

### Start Nimbus / UI Servers
pdsh -h m2-denali.hwx.test 'sudo -u storm nohup storm nimbus &'
pdsh -h m2-denali.hwx.test 'sudo -u storm nohup storm ui &'

### Start Supervisors
pdsh -g denali/u 'sudo -u storm nohup storm supervisor &'

### Cleanup and try again
pdsh -g denali/storm 'rm -rf /etc/storm /usr/share/storm-0.9.1;rm /usr/bin/storm'

### Update the Configurations
pdsh -g denali/storm 'cd /etc/storm/conf;git pull' | dshbak

Will require a restart.