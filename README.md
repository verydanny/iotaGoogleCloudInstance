## Getting IRI running on a Google Cloud Instance

Google Cloud gives you $300 in free credit once you join, so that's a node running
for free for at least 3 months.

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

1. 
