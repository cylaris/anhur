# Anhur IMIE [Isolated Malware Investigation Environment].

Please note: If you decide to proceed in following this tutorial in building your own sandbox, you accept that:
Software used in the making of this behemoth is not owned by the repository owner, it is the proprietary software of their respective owners unless its licensing states otherwise. 
Absolutely NO CRYING if you fail to isolate your environments, I take no responsibility in this as I have not and will not audit individual environments, if you are not sure if you are able to isolate effectively or you think that my precautions are not enough, either a). stop right here or b). add more security precautions and carry on.

1. You will need some virtual machines to be the 'sandbox' essentially a sandbox is nothing special, it is purely a windows machine that is segregated from the rest of the network. So create some virtual machines wherever you have space for them. A RAM of 4GB/Machine is recommended though I'm sure you could run it on 2GB if you REALLY wanted to...
(I found the easiest way to grab an ISO for Windows 10 was via https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-10-enterprise - 90 days Win 10 enterprise eval edition, the time should reset if you are reverting the machine, if not you can use CMD and type 'slmgr -rearm', without quotes, to reset the 90 days.)

2. Ensure that each of these machines has an internet connection but is also isolated from any production environments, you can use
the machine to see if you can ping the e.g. 192.168.30.0/24 production range. I used a seperate vm to host a simple IP Fire firewall and re-routed all of my traffic via that, allowing all -> deny 10.* or 192.168* or 172.16.0.0 - 172.31.255.255, then drop a rule below this to allow PC's, both in and out, on internal net you will be using to interact with the sandboxes). IP Fire firewall download link: https://www.ipfire.org/

3. Build an Elastic Stack: this can be done by following a guide such as this one: https://logz.io/learn/complete-guide-elk-stack/ you
can of course always follow Elastic's own guides but I have always found third party guides better.

4. Once you are sure that these machines are segregated, you will need to install Suricata and Filebeat; Suricata to generate logs, alerts
and Filebeat to ship these alerts to the Elastic Stack, configure Filebeat to pick up the logs at c:\program files\suricata\log\eve.json,
and also configure it to send the logs to your Elasticsearch instance (e.g. http://192.168.30.30:9200), authentication is really only necessary externally.
Suricata download link: https://suricata-ids.org/download/
Filebeat download link: https://www.elastic.co/downloads/beats/filebeat

5. Don't forget once the install for those are complete, using the NSSM (Non-sucking service installer) install this command to be
run as a service, using the startup directory as Suricata, the Program to use Powershell and the latter, NSSM: 
/usr/local/bin/suricata -c /usr/local/etc/suricata/suricata.yaml -i {network interface; for example eth0 or wlan0 but these can vary} this will bind Suricata to your network interface to monitor and allow it to log, and when run via NSSM => powershell, will create a service allowing it to start the service on every reboot, like when you revert the VM after log out etc.
NSSM download link: https://nssm.cc/download

6. On your hypervisor configure autorevert using such a command: vim-cmd vmsvc/snapshot.revert 6 1 PowerOn (where 6 is the VM ID and 1 is the snapshot ID) this can be done on disconnect from a VNC session for example, leaving it effortless for you.

7. You're no where near done so don't flap now, we've got to create the web-based Websockify (websocket) & Nginx power combo. To do such a thing it is recommended to watch a tutorial such as this: https://www.youtube.com/watch?v=NYEyyVDsapw this needs to be done for each sandbox you're gonna use. The lady in this video is definitely not gonna help your nerves because she has as much idea as you do about creating it but nails it in the end. Sorry.

8. As it is covered in the video you watched at the beginning of this tutorial, you will need to ensure that NoVNC is running correctly on the sandbox, to do this you should visit http://{sandbox ip}, this is what will allow authentication into your environment. Ensure within the settings -> advanced the same IP address and port 8000, or whatever port your websockify is going to be running is in there. That should allow you to connect. 
As previously mentioned, the video referenced by this tutorial covers this, and Nginx should be hosting NoVNC.
NOVnc donwload link: https://github.com/novnc/noVNC/releases
Nginx download link: http://nginx.org/en/download.html

9. Once you're nice and burned out from having to sift through all the mixed signals given off in that video, you'll have to get node.js/ NPM. That's if she didn't cover it in the video, I can't bring myself to watch it again so if she did ignore this bit. Node.js basically runs Websockify.js to then proxy from Port 8000 or whatever random one you chose, to 5900 VNC port. Use NPM install to grab an app called forever. Forever will make Websockify run FOREVER. Use Forever 1.0.0 otherwise you damn well better get it right first time or it turns sentient and never stops. EVER.
NodeJs download link: https://nodejs.org/en/
NPM download link: https://www.npmjs.com/ 
NPM command to grab Forever 1.0.0: npm i forever@1.0.0

10. Download something like Git Bash to allow yourself to run a shell to then run websockify, you cant just do it via Node / Forever, I tried it. Bash will allow you to create a shell to run it, if you configure that properly via NSSM as well, you'll get websockify.js ran via Node.js Forever.js and then you're laughing because step 8/9 are savage.
GitBash download: https://git-scm.com/downloads

11. It may be worth now going back and making sure all the rules you want are in Suricata, in C:\Program Files\Suricata, PTSecurity offer some good open-source rules as well as EmergingThreats which are preinstalled. Don't forget about the free ET Pro rules using ET Telemetry.
You can build your own rules with something like snorpy if you've not even built them before.
ET Pro Telemetry: https://docs.opnsense.org/manual/etpro_telemetry.html
ET Pro (these will cost): https://www.proofpoint.com/uk/products/et-pro-ruleset
PTSecurity Free: https://github.com/ptresearch/AttackDetection
Open-source rules: https://github.com/suricata-rules/suricata-rules

12. You should create seperate Alert Signatures/ DNS Query visualizations through Kibana filtered out based on host.name for each host, you should also click the small calandar on the search bar, and at the bottom add auto refresh every 1/2 sec or whatever your preference to make the visualization auto refresh when it is an iFrame, this will be exported with the saved object.

13. You can make a secondary webserver to host all the seperate VM's with all their seperate Nginx sandbox pages, I did this. Once you  eventually cave to the same pressure you can use the HTML document in the master of /not-a-sandbox as it holds the iFrames for the VM, alerts, connection logs.

14. iFrames! You can pull these out of Kibana to put them where I've put placeholders in the HTML by going to the top left of your
        visualization, Share, Embed Code, Saved           Object, Copy iFrame code. If you're sinful enough to ignore this, and click short URL or the other option, it will just give you a saved version, and you wont see any new logs, the saved object is interactive but it has a very
long URL.

15. It might be worth, if your ElasticSearch isnt working, using the Elasticsearch.yml found in the master directory of this commit. It is also recommended to change the following from -Xms1g to -Xms4g within jvm.options in /etc/elasticsearch, this stops the dying elasticsearch from the constantly autorefreshing visualizations.

"# Xms represents the initial size of total heap space
“# Xmx represents the maximum size of total heap space

-Xms4g
-Xmx4g"

16. I guess once you've done that you've got a working sanbox hub. Congrats. Any questions reach out. I will be adding sources links now. 
