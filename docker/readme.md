# Start Klipper, Fluidd, Mainsail, Octoprint with Streaming Cam and Config Editor and also a firmware build image

##  Startup via docker-compose (or e.g. stacks in portainer)
I like portainer as docker Web-GUI. You can use it to start the yaml file as a stack after bringing portainer up by:  
```docker run -d -p 9000:9000 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce```
  
Access Portainer:  http://192.168.178.120:9000

## Quick-Start for Qidi X-Plus: 
   1. Start the stack (see yaml file in this folder)
   2. Access the Config-Editor (see the links below) and upload all files found in
           https://github.com/naxan6/QidiKlipperFlashConfig/tree/main/configs 
      to 
           /cfg     
      (drag-and-drop from eg. windows explorer works great!)
   3. Alter printer.cfg to match your printers serial-device
   4. Right click Folder "cfg" and select "open in integrated terminal"
       --> the terminal open in the lower part.
   5. execute 
            chmod 666 moonraker.cfg
      --> this grants readwrite files to everyone (including the moonraker-software that needs this)
   4. Restart the containers mhprinddklipper and after that mhprinddmoonraker
   5. Access Fluidd-Webgui (and all others) and check connectivity/configure everything to your liking (see the links below)

# Links to all the apps (you have to use your own IP!)
  * Fluidd:          http://192.168.178.120:8093
  * Mainsail:        http://192.168.178.120:8094
  * Octoprint:       http://192.168.178.120:8095

  * Config-Editor:   http://192.168.178.110:8096/?folder=/mhprinddFolders/

  * Cam1-Stream:     http://192.168.178.120:8093/cam1/stream         (the other ports 8094 and 8095 should also work)
  * Cam1-Snapshot:   http://192.168.178.120:8093/cam1/snapshot       (the other ports 8094 and 8095 should also work)
