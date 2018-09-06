# Install the packages
# Java and Kafka

sudo yum install -y java-1.8.0-openjdk-headless
sudo rpm -ivh kafka-1.1.0-0.x86_64.rpm

Set up the LVM volume group
The below is based on 4 x 10TB volumes attached to be configured as LVM striped volume group.

Firstly initialize the physical volumes so they are visible to LVM:

sudo pvcreate /dev/vdb /dev/vdc /dev/vdd /dev/vde
Next create a volume group containing the above devices

sudo vgcreate datavg /dev/vdb /dev/vdc /dev/vdd /dev/vde
Create a LVM striped logical volume using 100% of the available space in the volume group. 
We use 4 stripes as we have 4 devices:

sudo lvcreate --stripes 4 -l 100%FREE --stripesize 256 datavg -n lv_data
Next format the above logical volume as XFS:

sudo mkfs -t xfs /dev/mapper/datavg-lv_data
Next add a mount entry to /etc/fstab:

/etc/fstab
/dev/mapper/datavg-lv_data /data xfs defaults 0 1
Create the mount directory

sudo mkdir /data
Mount the volume

sudo mount -a
Crate the kafka directory and set correct permissions:

sudo mkdir /data/kafka
sudo chown kafka:kafka /data/kafka
sudo chmod 2775 /data/kafka
Get Kafka server.properties and sanitise permissions
Automation of download of server.properties TODO

sudo chown root:root /etc/kafka/server.properties
sudo chmod 0644 /etc/kafka/server.properties

Get  truststore/keystore and sanitise permissions
Automation of download of truststore and keystore TODO

sudo mv iag.truststore.jks /etc/ssl
sudo chmod 0444 /etc/ssl/iag.truststore.jks
sudo chown root:root /etc/ssl/iag.truststore.jks

# Set this to your hostname
# e.g. kafka00.com
sudo mv kafka00.com.keystore.jks /etc/ssl
sudo chmod 0400 /etc/ssl/kafka00.com.keystore.jks
sudo chown kafka:kafka /etc/ssl/kafka00.com.keystore.jks

Enable authentication in server.properties
ssl.endpoint.identification.algorithm=HTTPS

Enable and turn on Kafka
sudo systemctl enable kafka.service
sudo systemctl start kafka.service

journalctl -u kafka.service
journalctl -u kafka.service -f
