# -PlexWindowsGrafana

So, here's a tiny walkthrough of what I did to make it work. Please note I'm not a programmer, or a very good writer out of instructions, but I'll do my best. Nor do I really know what I'm doing. Also, I tend to ramble a lot when I write out instructions, so sorry in advanced that this is going to be long. I'll try make it worth it by talking mostly about what's going on and what's needed, as well as some troubles and issues I had, and how I got around them, but you've been warned it will be long. Sorry.

**THIS ISN'T GOING TO BE A FULL ON WALKTHROUGH** (because I *really* don't know what I'm doing), but hopefully some good steps on how to get there! I did this all on Windows 10, and also used some docker containers. 

Also, since I only found out about all this like a week ago and don't really know that much about it, while I may be able to help with small issues, the best course of action is to perhaps check out the appropriate subreddit, site or discord for most of these services (with an extra special mention for the guys at the /r/homelab discord, who were amazingly helpful and patient with my Grafana issues towards the end!)

Firstly, you'll need to get [Organizr](https://organizr.app/) set up. It's an absolutely amazing program that lets you run all of your tabs (like all the *arrs, Plex, Tautulli, and basically anything else on your network that's behind a localhost:port type address) in one tab without (mostly) leaving the window. Even if you don't want the fancy graphs and stats, Organizr is still amazing and I highly recommend you check them out!

When adding things to Organizr, you can add them as a new tab or as a homepage item. I recommend going through the tutorial quickly to make sure you know the difference, because what you see in my picture is my homepage, whereas all the tabs for things I can access are on the left

Next I went and got [docker](https://www.docker.com/) running. This will vary from system to system, so I'll let you handle that part yourselves. 

I decided I wanted [Monitorr](https://github.com/Monitorr/Monitorr), so I went and installed that as my first docker container. This lets me see if any of my services (Plex, Tautulli, *arrs etc) are online or offline. Takes a bit of tweaking, but I got there in the end. I got Monitorr connected to Organizr, and added in the Monitorr option on the homepage. 

Then I wanted a speed test type thing to see how the internet's doing. I found [this](https://github.com/henrywhitaker3/Speedtest-Tracker) one that works pretty good and also used docker to install it. I connected SpeedTest up to Organizr too and also added that to the homepage.

Next, I went to this page [here](https://technicalramblings.com/blog/spice-up-your-homepage-part-ii/). I tried to install things without using docker and had a mild panic attack, so decided to follow a guide to install using docker instead. I basically went to the [Varken site](https://github.com/Boerderij/Varken) and followed the instructions on the [wiki here](https://wiki.cajun.pro/books/varken/chapter/installation) to install with a docker container. One thing I'll say now that really helped me (thanks to the guys at the /r/homelab discord) - this tutorial makes an internal network for Varken, InfluxDB and Grafana to run on, which means later if you want to set up CPU/RAM/HDD and download/upload speed using Telegraf, you may need to allow InfluxDB to talk outside the docker container. Basically, once you get to step 2 of the docker install guide ([here](https://github.com/Boerderij/Varken/blob/master/docker-compose.yml)), it'll ask you to save the docker file. Make the changes needed, and also go to the services section, then influxdb, and anywhere you want add the following lines:

    ports:
          - 8086:8086

This made the InfluxDB part of my docker-compose.yml file look like this:

      influxdb:
        hostname: influxdb
        container_name: influxdb
        image: influxdb:1.8
        networks:
          - internal
        ports:
          - 8086:8086
        volumes:
          - ./opt/dockerconfigs/influxdb:/var/lib/influxdb
        restart: unless-stopped

A couple of side notes while we're here. You see how Varken won't work with InfluxDB 2, and you need version 1.8 to make it work? That line where I have "image: influxdb:1.8" is the part of this code that says to download version 1.8. If you don't have that, it'll automatically download the latest version for you, so make sure you add the :1.8 to the end of your image line. 

Second side note: when you get to step 7 where you stop Varken running and change the config for it, you also need to put all your API and server IP addresses and whatnot in the docker-compose.yml file too - didn't find that out until way later. Step 7 will ask you to edit the config file with things like how many different Sonarr, Tautulli, Radarr etc services are running, the API key for them, and their localhost:port type link. When that's all done, make sure you update the docker-compose.yml file with the same information, and then re-run the docker-compose command)

OK, back to the main guide. Once Varken, InfluxDB and Grafana were installed and working, I added the official Varken dashboard found [here](https://wiki.cajun.pro/books/varken/page/grafana#bkmrk-dashboard-0). Make sure you save your own dashboard as a copy. Once that's up and running, it'll start showing some stats for things like your Tautulli library and whatnot. I then added Grafana to Organizr's home page and as a tab.

Next step: installing Telegraf. Honestly, this step and the last step were where I had the most trouble, and though of giving up multiple times. It is not easy, nor is there a lot of helpful information out there on how to make it work properly. As I've said before, this is where the guys at the /r/homelab discord helped me the most, and without them I wouldn't have the dashboard I have now. I eventually decided to install Telegraf not through Docker because we started hating each other, so just did it the old fashion install way and followed the instructions on their [website](https://docs.influxdata.com/telegraf/v1.20/introduction/downloading/). Note, there is an option to install as a Docker container, but as mentioned, it didn't work for me so I gave up on Docker and got it as a Windows Binary. I followed their way to set things up. I extracted it to C:/Program Files/Telegraf. Now, on the website it says to have a conf folder within that folder, and an input.conf and output.conf and whatnot. I don't recommend that. At least on Windows for me, I just have a telegraf.conf file in my C:Program Files/Telegraf folder and that's it.

Now for the telegraf.conf, I found [this Grafana dashboard](https://grafana.com/grafana/dashboards/1902) that has some info on linking Telegraf to Grafana to get the RAM/CPU/HDD usage. They have a list of what things should be available and uncommented to get the dashboard working, so go through and uncomment everything they say. Now that's all the "Input Plugins" done for Telegraf, you need to do the output plugin. I deleted everything in the OUTPUTS section because it was set up for InfluxDB v2 (which we didn't use, remember?), and replaced it with everything you need for InfluxDB v1:

    ###############################################################################
    #                               OUTPUTS                                       #
    ###############################################################################
    
    [[outputs.influxdb]]
      urls = ["http://localhostip:8086"]
      database = "telegraf"

Change that URL as needed - for me it was a fun 192.168.0 address. You can make sure it's working by running the command on the Telegraf website, and if it's working with no errors, then yay! Congrats! Last step is now putting it all together.

I used that [Telegraf/Grafana dashboard](https://grafana.com/grafana/dashboards/1902) from before and imported that into Grafana too. Now that I had 2 different dashboards, I took the best of both of them (i.e. the parts that I wanted) and put them together. I made a dashboard that I liked, and went back to [this site](https://technicalramblings.com/blog/spice-up-your-homepage-part-ii/) where they talk about getting it working with the home page. You basically want to choose a Custom HTML homepage item in Organizr and paste in the code on the website. If you go to your Grafana dashboard, you'll see that you can click on a panel and choose to share it. If you go to the embed tab and change untick the option for "Current time range", you'll get a url that you can use in the website's tutorial. If you want your dashboard to look how it is on the site, keep doing this with all the panels you want, and that's it! If you want the same sort of look as what mine in, [here's a gist](https://gist.github.com/Opaque02/ad50fa82c43fe5cc2021d6b98d1b4dfa) of the custom HTML/CSS code I made that you can copy and paste in, changing the links as needed. Have a play around with the code as well if you want! Also, this works on mobile!

After that, that's pretty much it! Sorry again this is so long, but hopefully this will help a few people who got stuck on the same places I did!

Also, I don't know if it'll work, but [here's my Grafana dashboard](https://github.com/Opaque02/-PlexWindowsGrafana) ready to be imported into Grafana. It may require changing the name of some things here and here (like your server name and whatnot potentially?), but if you have the same things set up as what I went through in this guide, hopefully you'll just need minor tweaks, and then the HTML/CSS will just need you to change the links for your specific IP and Grafana ID. I've not done this before so I don't know how it works. But in theory once you import it, just change the details to yours, add the custom HTML/CSS to Organizr and it should be up and running! Good luck, and hope it works well for you!
