# Install the packages
# Java and Kafka

<br>sudo yum install -y java-1.8.0-openjdk-headless
<br>sudo rpm -ivh kafka-1.1.0-0.x86_64.rpm

<h3>Set up the LVM volume group</h3>
<p>
The below is based on 4 x 10TB volumes attached to be configured as LVM striped volume group.
Firstly initialize the physical volumes so they are visible to LVM:
</p>  

<br>sudo pvcreate /dev/vdb /dev/vdc /dev/vdd /dev/vde
<h3>Next create a volume group containing the above devices</h3>

<br>sudo vgcreate datavg /dev/vdb /dev/vdc /dev/vdd /dev/vde

<p>
  Create a LVM striped logical volume using 100% of the available space in the volume group. 
We use 4 stripes as we have 4 devices:
</p>  

<br>sudo lvcreate --stripes 4 -l 100%FREE --stripesize 256 datavg -n lv_data

<h3>Next format the above logical volume as XFS:</h3>

<br>sudo mkfs -t xfs /dev/mapper/datavg-lv_data

<h3>Next add a mount entry to /etc/fstab:</h3>

/etc/fstab
/dev/mapper/datavg-lv_data /data xfs defaults 0 1

<h3>Create the mount directory</h3>

<br>sudo mkdir /data
<br>Mount the volume

<br>sudo mount -a
<h3>Crate the kafka directory and set correct permissions:</h3>

<br>sudo mkdir /data/kafka
<br>sudo chown kafka:kafka /data/kafka
<br>sudo chmod 2775 /data/kafka

<h3>Get Kafka server.properties and sanitise permissions</h3>
Automation of download of server.properties TODO

<br>sudo chown root:root /etc/kafka/server.properties
<br>sudo chmod 0644 /etc/kafka/server.properties

<h3>Get  truststore/keystore and sanitise permissions</h3>
Automation of download of truststore and keystore TODO

<br>sudo mv iag.truststore.jks /etc/ssl
<br>sudo chmod 0444 /etc/ssl/iag.truststore.jks
<br>sudo chown root:root /etc/ssl/iag.truststore.jks

<h3> Set this to your hostname </h3>
<h5> e.g. kafka00.com</h5>
<br>sudo mv kafka00.com.keystore.jks /etc/ssl
<br>sudo chmod 0400 /etc/ssl/kafka00.com.keystore.jks
<br>sudo chown kafka:kafka /etc/ssl/kafka00.com.keystore.jks

<h3>Enable authentication in server.properties</h3>
<br>ssl.endpoint.identification.algorithm=HTTPS

<h3>Enable and turn on Kafka</h3>
<br>sudo systemctl enable kafka.service
<br>sudo systemctl start kafka.service

<br>journalctl -u kafka.service
<br>journalctl -u kafka.service -f
