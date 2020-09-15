![alt text](https://i.ibb.co/nPpksv0/basedfir-final-illustration-x4-colored-toned-illustration-x2.png)

## Anhur IMIE | project by BaseDFIR ##
This is a project based on various other softwares and tools, it is likely effective in an internal environment as well as most SOC's as it provides a good level of capability in sandboxing malware, you can at your own discretion add further tools into the VM you wish to use for analysis, such as dissassemblers or dynamic analysis tools to further investigate. 

Elastic Kibana Suricata ECS Events:
![alt text](https://www.elastic.co/guide/en/beats/filebeat/current/images/filebeat-suricata-events.png)

Please feel free to add to the project by adding in regmon/ fim/ sysmon. These additions will be coming however we are working on polishing off the first edition prior to bringing further ingredients to the mix.

## Please note: ##
If you decide to proceed in following this tutorial, you accept that:

Software used in the making of this behemoth is not owned by the repository owner, it is the proprietary software of their respective owners unless its licensing states otherwise. 

If you fail to isolate your environments, I take no responsibility in this as I have not and will not audit individual environments, if you are not sure if you are able to isolate effectively or you think that my precautions are not enough, either a). stop right here or b). add more security precautions and carry on.

There is currently not a completed version of the installation script available for download however this will be coming in the near future, so if you'd prefer to wait then do that as I will need infrastructure to develop this on. (If you can provide this please contact me - thank you!).

## Steps ##
1. Windows 10 VM spun up, eval edition can be found here: https://www.microsoft.com/en-gb/evalcenter/evaluate-windows-10-enterprise - this lasts 90 days. If not you can use CMD and type 'slmgr -rearm' to reset it.


2. Install Filebeat on Win 10 VM; configure filebeat to pull logs from c:\program files\suricata\log\eve.json and then configure it to send the logs to your Security Onion instance when it is configured (e.g. http://192.168.0.1:9200), authentication is really only necessary externally.


3. On your physical host, configure autorevert, this can vary depending on your architecture:
VMWare:

Hyper-V:

Citrix:

Red Hat:

KVM:


4. Configure TightVNC, to forward connections from a websocket you will set up in the next step:
TightVNC acts as a VNC server. In this set-up, websockify (a websocket-to-TCP-bridge) is to proxy a port from the web, to the VNC client, and NoVNC's purpose is to purely be the web-interface.


5. Configure Nginx, NoVNC and Websockify. These are all together for a reason:

    a). **Install NPM**. 
    
    b). Test installation is working: 
       
        npm -v
    
    c). **Install Websockify** (websocket): 
    
        npm install --save node-websockify

    d). **Create a folder** anywhere called: '*NoVNC*'
    
    e). A **folder within the NoVNC folder** called: '*Websockify*'
    
    f). **Within the Websockify folder**, using notepad, **save a document there named: websockify.js** (ensure to save it as all files type, not txt).
    
    g). **Paste this into websockify.js** that was just created: e.g. 
        
        var websockify = require('node-websockify');
        websockify({  source: '127.0.0.1:8080', target: '192.168.0.100:5900'});
        
    h). **Create a folder in the root folder**, NoVNC named '**Nginx**' and then **extract the contents of Nginx-(ver).zip here**
    
    i). **In the HTML folder, remove what is there. Then download the NoVNC files (https://github.com/novnc/noVNC.git) into the directory (/NoVNC/Nginx/HTML/)**
    
    j). Start nginx, **go to the /NoVNC/Nginx/ directory within CMD and type**:
    
        start Nginx
    
    k). Test this is all working by going to *localhost/vnc.html* (if you see NOVNC in large green/yellow letters - it is!).
    
    l). Within the NoVNC page you have open now in your browser, **go to settings -> advanced -> websocket the host should stay 'localhost' for now, on another PC you will have         to type this small bit again with its actual IP address (ipconfig), the port is 8080 (as used in my example) and the path should stay websockify**.
    
Once you have typed the config settings in websocket, it will work for that PC. You will need to do this small part on all PC's you plan to use it on. Externally of it luckily it will have the same IP address. If you follow until the end and configure a seperate nginx-hub to host multiple, this will become further seamless.


6. Download something like Git Bash to allow yourself to run a shell to then run websockify, you cant just do it via Node / Forever, I tried it. Bash will allow you to create a shell to run it, if you configure that properly via NSSM as well, you'll get websockify.js ran via Node.js Forever.js and then you're laughing because step 8/9 are savage.

Download GitBash to make an easy script, that will run websockify.js each time the VM is started. It will utilise a module of NPM called forever. The most stable version is 1.0.0:

NPM command to grab Forever 1.0.0: npm i forever@1.0.0

GitBash can be downloaded from this repository.

7. You can make a secondary webserver to host all the seperate VM's with all their seperate Nginx sandbox pages, I did this. Once you  eventually cave to the same pressure you can use the HTML document in the master of /not-a-sandbox as it holds the iFrames for the VM, alerts, connection logs.

8. iFrames! You can pull these out of Kibana to put them where I've put placeholders in the HTML by going to the top left of your visualization, Share, Embed Code, Saved        Object, Copy iFrame code. 

9. It might be worth, if your ElasticSearch is running slowly, using the Elasticsearch.yml found in the master directory of this commit. It is also recommended to change the following from -Xms1g to -Xms4g within jvm.options in /etc/elasticsearch, this stops the dying elasticsearch from the constantly autorefreshing visualizations.

"# Xms represents the initial size of total heap space
“# Xmx represents the maximum size of total heap space

-Xms4g
-Xmx4g"

# Hello World! ttfhw? - no?
If you have carried out these steps effectively it is likely that you have a working sandbox environment, in the future I plan to add further capability to this project including adding a fake network feature to force malware to disclose its IOC's without using a dissassembler as well as anti-evasion techniques. However prior to this I will get myself a development environment where I can fully flesh out this tutorial with images and possibly a POC video so that it is much easier to follow along. You can obviously create this same VM with other OS's or add in a lot of other capability as discussed above.

If you would like to contribute don't hesitate to reach out to me, it would be good to have some helping hands and possibly build some other cool stuff. 
