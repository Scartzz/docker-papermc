## Minecraft server PAPERMC on Ubuntu 16.04 with openjava 1.8

This docker image builds and runs the newest papermc version of minecraft. 

If the paper.jar is not found in the minecraft directory the system pulls down the Repo from PaperMC and build a new paper.jar from the
released minecraft.jar

Each time the container is started the presence of the file /minecraft/paper.jar, if the file is missing a build of paper.jar is started.

The paper daemon is started with superovisord, see nimmis Ubuntu container for a more detailed description of nimmis implementation of an init-process in ubuntu, see [nimmis/ubuntu](https://hub.docker.com/r/nimmis/ubuntu/)

Whats new is

- Autodetection of timezone if container has access to internet
- adjust minecraft user UID to match mounted volume
- selectable memory size for the Java process
- do a nice shutdown of the server when the docker stop command is issued
- docker accessible commands to 
   - start/stop/restart the papermc server
   - send console commands to the server
   - look at console output from the server
- 

## Starting the container

To run the latest stable version of this docker image run

	docker run -d -p 25565:25565 -e EULA=true scartzz/docker-papermc

the parameter

	-e EULA=true

The is because Mojang now requires the end user to access their EULA, located at
https://account.mojang.com/documents/minecraft_eula, the be able to start the server.

the parameter

	-p 25565:25565

tell on witch external port the internal 25565 should be connected, in this case the same, if
you only type -p 25565 it will connect to a random port on the machine

## Giving the container a name

To make it easier to handle you container you can give it a name instead of the long
number thats normally give to it, add a

	--name papermc

to the run command to give it the name minecraft, then you can start it easier with

	docker start papermc
	docker stop papermc

## First time run

This will take a couple of minutes depending on computer and network speed.

you can follow the output from the compilation with then command (assume that you given the container
the name papermc)

	docker logs -f papermc
    *** open logfile
    *** Run files in /etc/my_runonce/
    *** Running /etc/my_runonce/set_timezone...
    *** Run files in /etc/my_runalways/
    *** Running /etc/my_runalways/do_build_paper...

Then the compilation is completed the server will start and you will see something like

    *** Running /etc/my_runalways/eula...
    *** Running /etc/rc.local...
    *** Booting supervisor daemon...
    *** Supervisor started as PID 4820
    *** Started processes via Supervisor......
    crond                            RUNNING    pid 4824, uptime 0:00:03
    paper                            RUNNING    pid 4825, uptime 0:00:03
    syslog-ng                        RUNNING    pid 4823, uptime 0:00:03


you can then exit from the log with CTRL-C

### setup memory to use

There are two environment variables to set maximum and initial memory for papermc.

#### MC_MAXMEM

Sets the maximum memory to use <size>m for Mb or <size>g for Gb, if this parameter is not set 1 Gb is chosen, to set the maximum memory to 2 Gb

    -e MC_MAXMEM=2g

#### MC_MINMEM

sets the initial memory reservation used, use <size>m for Mb or <size>g for Gb, if this parameter is not set, it is set to MC_MAXMEM, to set the initial size t0 512 Mb

    -e MC_MINMEM=512m


#### PAPER_AUTORESTART

This variable controlls the behavior of the container when the **stop** command is issued inside minecraft

	-e PAPER_AUTORESTART=yes
   
Which is the default behavior and does not need to be specified, the minecraft server will autostart if the **stop** command is issued.

	-e PAPER_AUTORESTART=no
	
If the **stop** command is issued the minecraft server will stay down until the container is restarted or the command **mc_start** is issued

## look at the last output from the papermc server

To get an output of the latest events from the papermc server type

	docker exec papermc mc_log

and you will see the last 10 lines from the output, this is what you will see after startup

	Abort with CTRL-C
	[13:02:15 INFO]: Zombie Aggressive Towards Villager: true
	[13:02:15 INFO]: Experience Merge Radius: 3.0
	[13:02:15 INFO]: Preparing start region for level 0 (Seed: 506255305130990210)
	[13:02:16 INFO]: Preparing spawn area: 22%
	[13:02:17 INFO]: Preparing spawn area: 99%
	[13:02:17 INFO]: Preparing start region for level 1 (Seed: 506255305130990210)
	[13:02:18 INFO]: Preparing spawn area: 95%
	[13:02:18 INFO]: Preparing start region for level 2 (Seed: 506255305130990210)
	[13:02:18 INFO]: Server permissions file permissions.yml is empty, ignoring it
	[13:02:18 INFO]: Done (3.650s)! For help, type "help" or "?"

It will continue to output everything from the console until you press CTRL-C 

## sending commands to the server console

You don't need to have an interactive container to be able to send commands to the console. To send
a command to set the time to day you type

	docker exec papermc mc_send "time set day"

If this was the first command issued after a start the output should look like

	[13:02:15 INFO]: Zombie Aggressive Towards Villager: true
	[13:02:15 INFO]: Experience Merge Radius: 3.0
	[13:02:15 INFO]: Preparing start region for level 0 (Seed: 506255305130990210)
	[13:02:16 INFO]: Preparing spawn area: 22%
	[13:02:17 INFO]: Preparing spawn area: 99%
	[13:02:17 INFO]: Preparing start region for level 1 (Seed: 506255305130990210)
	[13:02:18 INFO]: Preparing spawn area: 95%
	[13:02:18 INFO]: Preparing start region for level 2 (Seed: 506255305130990210)
	[13:02:18 INFO]: Server permissions file permissions.yml is empty, ignoring it
	[13:02:18 INFO]: Done (3.650s)! For help, type "help" or "?"
	[13:12:35 INFO]: Set the time to 1000

It will continue to output everything from the console until you press CTRL-C

### using the minecraft op command 

To make yourself operator in the game use **mc_send** command, for example give the user **myuser** op use the command.

	docker exec papermc mc_send op myuser

### using the minecraft stop command

Default the minecraft server will automatically restart on a **stop** inside the minecraft application. You can override this behavior by using

	-e PAPER_AUTORESTART=no

This will prevent the server to restart and minecraft has to be started again with the **mc_start** command


## starting and stopping the server

To stop the server but not the container do

	docker exec papermc mc_stop

To start it after being stopped do

	docker exec papermc mc_start

Finally to restart it do

	docker exec papermc mc_restart

## stopping the container

When the container is stopped with the command

	docker stop papermc

the papermc server is shutdown nicely with a console stop command to give it time to save everything before
stopping the container. If you look in the output from the server this show

	[13:01:51 INFO]: Stopping the server
	[13:01:51 INFO]: Stopping server
	[13:01:51 INFO]: Saving players
	[13:01:51 INFO]: nimmis lost connection: Server closed
	[13:01:51 INFO]: nimmis left the game.
	[13:01:51 INFO]: Saving worlds
	[13:01:51 INFO]: Saving chunks for level 'world'/Overworld
	[13:01:51 INFO]: Saving chunks for level 'world_nether'/Nether
	[13:01:51 INFO]: Saving chunks for level 'world_the_end'/The End

## Having the minecraft files on the host machine

If you delete the container all your filer in minecraft will be gone. To save them where it's
easier to edit and do a backup of the files you can attach a directory from the host machine
(where you run the docker command) and attach it to the local file system in the container.
The syntax for it is

	-v /host/path/to/dir:/container/path/to/dir

To attach the minecraft directory in the container to directory /home/nimmis/mc-srv you add

	-v /home/nimmis/mc-srv:/minecraft

### problems with external mounted volumes

When a external volume is mounted the UID of the owner of the volume may not match the UID of the minecraft user (1000). This can result in problems with write/read access to the files. 

To address this problem a check is done between UID of the owner of /minecraft and the UID of the user minecraft. If there is a mismatch the UID of the minecraft user is changed to match the UID of the directory.

If you don't want to do this and want to manually set the UID of the minecraft user there is a variable named PAPER_UID which defines the minecraft user UID, adding

	-e PAPER_UID=1132

sets the minecraft user UID to 1132.

## Issues

If you have any problems with or questions about this image, please contact me by submitting a ticket through a [GitHub issue](https://github.com/scartzz/docker-papermc/issues "GitHub issue")

1. Look to see if someone already filled the bug, if not add a new one.
2. Add a good title and description with the following information.
 - if possible an copy of the output from **cat /etc/BUILDS/*** from inside the container
 - any logs relevant for the problem
 - how the container was started (flags, environment variables, mounted volumes etc)
 - any other information that can be helpful

## Contributing

You are invited to contribute new features, fixes, or updates, large or small; we are always thrilled to receive pull requests, and do our best to process them as fast as we can.

## Future features

- automatic backup
- plugins
- more....

