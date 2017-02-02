VM for WebSphere MQ 7.5, using Ubuntu with Vagrant.

Using free developer version of WebSphere MQ.

## MQ Installation

VM: `vagrant up`

Installation medias:
https://www.ibm.com/developerworks/community/blogs/messaging/entry/develop_on_websphere_mq_advanced_at_no_charge?lang=en

Installing (on ubuntu):
https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.5.0/com.ibm.mq.ins.doc/q115250_.htm

### Detailed steps 

https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.5.0/com.ibm.mq.ins.doc/q008550_.htm

```
vi /etc/sysctl.conf

	fs.file-max = 524288

sudo sysctl -p
cat /proc/sys/fs/file-max

# install rpm
sudo apt-get install -y rpm bc

# confirm it is there
dpkg -l rpm

# unpack
mkdir /home/ubuntu/mq
tar xzvf /vagrant/mqadv_dev75_linux_x86-64.tar.gz -C /home/ubuntu/mq

sudo su -
cd /home/ubuntu/mq
./mqlicense.sh -text_only
rpm -ivh --nodeps --force-debian MQSeriesRuntime-*.rpm
rpm -ivh --nodeps --force-debian MQSeriesServer-*.rpm MQSeriesMan-*.rpm MQSeriesSamples-*.rpm MQSeriesJava-*.rpm

su mqm -c "/opt/mqm/bin/mqconfig"

echo 300 > /proc/sys/net/ipv4/tcp_keepalive_time

vi /etc/security/limits.conf

	* soft     nofile         65535   
	* hard     nofile         65535

su mqm -c "/opt/mqm/bin/mqconfig"

mkdir /usr/lib64
/opt/mqm/bin/setmqinst -i -p /opt/mqm

```

## Verify installation 

Source: https://www.ibm.com/support/knowledgecenter/en/SSFKSJ_7.5.0/com.ibm.mq.ins.doc/q009240_.htm

```
dspmqver
# status is returned

sudo su mqm

# Create a queue manager called QMAby entering the following command:
crtmqm QMA
# Messages indicate when the queue manager is created, and when the default IBM WebSphere MQ objects are created.
# Start the queue manager by entering the following command:
strmqm QMA
A message indicates when the queue manager starts.
# Start MQSC by entering the following command:
runmqsc QMA
# A message indicates when MQSC starts. MQSC has no command prompt.
Define a local queue called QUEUE1 by entering the following command:
DEFINE QLOCAL (QUEUE1)
# A message indicates when the queue is created.
# Stop MQSC by entering the following command:
end

cd /opt/mqm/samp/bin

# .. (follow the docs)
```

## Configs 

```
# disable auth
runmqsc QMA
ALTER QMGR CHLAUTH(DISABLED)
end

# run listener
runmqlsr -t tcp -p 1414 -m QMA &

runmqsc QMA
start listener(system.default.listener.tcp)
end

# create channel to send messages on
runmqsc QMA
DEFINE CHANNEL(MYCHANNEL) CHLTYPE(SVRCONN)
end
```

## Startup 
```
sudo su -

su mqm -c "strmqm QMA"

su mqm -c "runmqlsr -t tcp -p 1414 -m QMA &"
```

## Stop 

```
endmqm QMA
```

## Logs 

```
mqm@ubuntu-xenial:~/qmgrs/QMA/errors/*.LOG
```


