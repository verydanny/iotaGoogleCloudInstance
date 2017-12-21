## Getting IRI running on a Google Cloud Instance

Google Cloud gives you $300 in free credit once you join, so that's a node running
for free for at least 3 months.

**Table of Contents:**

[Starting Your Instance](#starting-your-instance)  
[Installing IRI](#installing-iri)  
[Monitoring IRI](#monitoring-iri)  

### Starting Your Instance

1. Make a Google Cloud account (https://console.cloud.google.com/).

2. Once created and logged in, you have to create a project.

<img src="./static/images/gc1.png" style="max-width: 450px">  

3. Once the box opens, create a new project "+", call it "Iota".

4. In the search box, type in `Instance`, and click on the result that says `Instances, Compute Engine`.

5. It'll take some time to start this service, but once it's started, click "Create".

6. Call the instance something like, "iota-full-node-1"

7. Pick a zone, if you're in the US, pick east, central, or west. The closer it is to you, the faster you can
ssh, etc. Here's the location of all specific zones: (https://cloud.google.com/compute/docs/regions-zones/)

8. Pick at least 2vCPU and 4gb or ram, recommended is 6gb+ and 4 cores, but that can get pricey.

9. Change your boot disk to Ubuntu 17.10, it's the latest and won't require a reboot. Make sure you give yourself at least
96gb of HD space (I recommend). Pick SSD for faster read/write or standard for cheaper but slower read/write.

10. **Important:** Under Identity and API access, make sure "Allow full access to all Cloud APIs" is checked.

11. **Important:** Under Firewall, make sure "Allow HTTP traffic" and "Allow HTTPS traffic" is checked.

12. Create instance

13. Once done, click the SSH icon, this will take you right to the shell where the real fun begins.

### Installing IRI

1. Lets upgrade as needed:
```bash
sudo apt update -qqy --fix-missing && sudo apt-get upgrade -y && sudo apt-get clean -y && sudo apt-get autoremove -y --purge
```  
2. We're going to install the Oracle version of Java 8, make sure you accept the license agreement.
```bash
sudo apt install software-properties-common -y && sudo add-apt-repository ppa:webupd8team/java -y && sudo apt update && sudo apt install oracle-java8-installer curl wget jq git -y && sudo apt install oracle-java8-set-default -y
```
3. Lets add 2 users, one for IOTA and one for nelson in the future. This is for security.
```bash
sudo useradd -s /usr/sbin/nologin -m iota
sudo useradd -s /usr/sbin/nologin -m nelson
```
4. Lets create directories where everything will live.
```bash
sudo -u iota mkdir -p /home/iota/node /home/iota/node/ixi /home/iota/node/mainnetdb
```
5. Lets download the latest IRI and put it in the node directory.
```bash
sudo -u iota wget -O /home/iota/node/iri-1.4.1.4.jar https://github.com/iotaledger/iri/releases/download/v1.4.1.4/iri-1.4.1.4.jar
```
6. We want to create a service for IRI, just in case you reboot or if you want to start it then stop it.
```bash
sudo nano /lib/systemd/system/iota.service
```
***IMPORTANT:*** Then paste, but remember to substitute `{{ SUBSTITUTE HERE }}` with your server's ram. Here's a simple chart:
- 4gb: `-Xmx3g`
- 6gb: `-Xmx4500m` or `-Xmx4g`
- 8gb: `-Xmx6144m` or `-Xmx6g`
- 10gb: `-Xmx8192m` or `-Xmx8g`
- 12gb: `-Xmx10240m` or `-Xmx10g`
- 16gb: `-Xmx14336m` or `-Xmx14g`
```bash
[Unit]
Description=IOTA (IRI) full node
After=network.target

[Service]
WorkingDirectory=/home/iota/node
User=iota
PrivateDevices=yes
ProtectSystem=full
Type=simple
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=60
ExecStart=/usr/bin/java {{ SUBSTITUTE HERE }} -Djava.net.preferIPv4Stack=true -jar iri-1.4.1.4.jar -c iota.ini
SyslogIdentifier=IRI
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
Alias=iota.service
```

7. Save with `Ctrl + X`, then `Y`, then `Enter`
8. To enable the service type:
```bash
sudo systemctl daemon-reload && sudo systemctl enable iota.service
```
You can not start/stop/restart the service with: `sudo service iota start|stop|restart|status`

9. Now we have to configure IRI.
```bash
cat << "EOF" | sudo -u iota tee /home/iota/node/iota.ini
```
Then paste the following, followed by enter:
```bash
[IRI]
PORT = 14265
UDP_RECEIVER_PORT = 14600
TCP_RECEIVER_PORT = 15600
API_HOST = 0.0.0.0
IXI_DIR = ixi
HEADLESS = true
DEBUG = false
TESTNET = false
DB_PATH = mainnetdb
RESCAN_DB = false
REMOTE_LIMIT_API="removeNeighbors, addNeighbors, interruptAttachingToTangle, attachToTangle, getNeighbors"
NEIGHBORS = udp://94.156.128.15:14600 udp://185.181.8.149:14600 udp://88.99.249.250:41041
EOF
```

10. Now we have to add a Database so IRI has a starting point for syncing. First we made (previously) the directory for the db, then we download the database inside. Lets navigate to the iota directory.
```bash
cd /home/iota/node/
```
Then download and copy the database into the directory, then delete the download (it's big, like 8gb). This will take some time.
```bash
sudo curl -O http://db.iota.partners/IOTA.partners-mainnetdb.tar.gz && sudo tar xzfv ./IOTA.partners-mainnetdb.tar.gz -C ./mainnetdb && sudo rm ./IOTA.partners-mainnetdb.tar.gz
```

11. We are ready to start the IRI node for the first time.
```bash
sudo service iota start
```

12. Lets enable auto-updating of the IRI if there's a new version. This will check for a new version every 15 minutes.
```bash
echo '*/15 * * * * root bash -c "bash <(curl -s https://gist.githubusercontent.com/zoran/48482038deda9ce5898c00f78d42f801/raw)"' | sudo tee /etc/cron.d/iri_updater > /dev/null
```
### Monitoring IRI

1. Check the health of the Node with this command, make sure you have no errors:
```bash
journalctl -u iota -f
```

2. Show all neighbors you have now:
```bash
curl http://localhost:14265 -X POST -H 'Content-Type: application/json' -H 'X-IOTA-API-Version: 1.4' -d '{"command": "getNeighbors"}' | jq
```

3. Show IRI Status:  
***PLEASE NOTE:*** It takes a long time to get up from milestone 243000 to the current one with default swarm nodes I provided. Nelson fixes this issues.
```bash
curl http://localhost:14265 -X POST -H 'Content-Type: application/json' -H 'X-IOTA-API-Version: 1.4' -d '{"command": "getNodeInfo"}' | jq
```
