
# Darwin Streaming Server 6.0.3  #

- setup, customization, plugin or module development, performance and load tests on 32 & 64 bit Redhat Linux

How to setup Darwin Streaming Server 6.0.3 on 32 or 64 bit Linux platforms, add custom functionality by developing plugins ("modules" as Apple calls them), and results of some performance and load tests I ran

Download stuff_u_need - 720.52 KB

## Introduction ##

Darwin Streaming Server is Apple's open source streaming server for the internet. This article will get you started with it, and if you choose to, take you to a certain depth regarding how to tinker with its code to add what you need. It has a very well supported modular architecture, which means you can add one or more custom module(s) - just dropping the binary in a certain folder is sufficient for the server to pick it up and use it. That's what I call plug an play. This is very useful if you do not want to re-build the server's code itself. This article will demonstrate exactly how to do that. Custom modules can be used to perform custom authentication/ authorization, custom logging, reporting, data collection or whatever you have in mind.

This article will also discuss the results of some load tests and performance tests I did on the server, trying to discover its limits. My platform was Redhat Linux, but the steps of setting up DSS are pretty much the same for most Unix platforms.

**Hinting**

Darwin Streaming Server can stream hinted media files, like mp4 or mov files - Apple formats that can be played back using the Quick Time Player. In fact, it can stream almost any other kind of content, as long as the file is hinted. You can "hint" media files using tools Apple provides. If you need to hint your media files, please research separately. I will assume that you have a catalog of hinted files which your Darwin server will stream to a Quicktime player (or any other player able to play Apple streams) running on some machine which can communicate with your server. The default installation of Darwin already comes with some sample hinted media files. To set up and test, you do not need any more files.

**Platform and Hardware**

**Server Platform**

    $ cat /etc/redhat-release

Red Hat Enterprise Linux AS release 4 (Nahant Update 6)

    $ gcc -v

```
Reading specs from /usr/lib/gcc/x86_64-redhat-linux/3.4.6/specs
Configured with: ../configure --prefix=/usr --mandir=/usr/share/man --infodir=/usr/share/info --enable-shared --enable-threads=posix --disable-checking --with-system-zlib --enable-__cxa_atexit --disable-libunwind-exceptions --enable-java-awt=gtk --host=x86_64-redhat-linux
Thread model: posix
gcc version 3.4.6 20060404 (Red Hat 3.4.6-9)
```

**Server Hardware**

```
Summary: Dell SC1425, 2 x Xeon 2.80GHz, 3.9GB / 4GB 400MHz
System: Dell PowerEdge SC1425 (Dell 0D7449)
Processors: 2 x Xeon 2.80GHz 800MHz FSB (HT enabled, 4 threads) - Nocona E0, 64-bit, 90nm, L2: 1MB
Memory: 3.9GB / 4GB 400MHz == 2 x 2GB - 2GB PC2-3200 Netlist DDR2-400 ECC Registered CL3 2R
4 x empty
Disk: sda (ata_piix0): 80GB (63%) JBOD == 1 x 80GB 7.2K SATA/300 WD Caviar SE 8MB
Disk-Control: piix_ide0: Dell/Intel ICH5 82801EB ATA/100
Disk-Control: ata_piix0: Dell/Intel ICH5 82801EB SATA/150
Chipset: Intel E7520 C1 (Lindenhurst), 82801E (ICH5)
Network: eth0 (e1000): Dell/Intel 82541PI Gigabit, *** masked MAC ***, 1Gb/s <full-duplex>
Network: eth1 (e1000): Dell/Intel 82541PI Gigabit, *** masked MAC ***, no carrier
OS: RHEL AS 4 U6, Linux 2.6.9-67.ELsmp x86_64, 64-bit
BIOS: Dell A02 08/23/2005
Hostname: **** masked hostname ****   
```

**Client **

For playback, I used a Windows box running Quicktime Player. For load testing, I used a different Linux box with profile similar to that of the server.

## Steps to download, install and setup DSS 6.0.3 ##

The following steps should work on both 32 bit and 64 bit Linux. Steps 3 and 4 use patch files that makes the DSS code build without errors on 64 bit. You can go ahead and apply the patches even if you are working on 32 bit, it is better to have portable code. On 64 bit, however, you must apply the patches, otherwise it won't build. 

It is possible, however, that Apple guys change the code on macosforge server without changing the version number, and then these patches might not work. I always encourage to use the --dry-run option with the patch command to make sure that the patch will actually go through before patching it without the option. 

Similarly, step 5 uses an "Install" script that is different from what will come with the download from macosforge server. The new install script has the necessary changes for 64 bit installation to go through nicely.

The 3 files that steps 3, 4 and 5 refer to are in the "Patches" directory of the source code.

1. Downloaded source tarball from http://dss.macosforge.org/ --> Darwin Streaming Server 6.0.3 --> Source code

2. `tar xf DarwinStreamingSrvr6.0.3-Source.tar` --> will create directory "DarwinStreamingSrvr6.0.3-Source"

3. `patch -p0 < dss-6.0.3.patch` - patch file included in uploaded source - patch makes it 64 bit compatible, run from directory containing DarwinStreamingSrvr6.0.3-Source

4. `patch -p0 < dss-hh-20080728-1.patch` - patch file included in uploaded source - patch makes it 64 bit compatible, run from directory containing DarwinStreamingSrvr6.0.3-Source

5. `cp Install DarwinStreamingSrvr6.0.3-Source` (new Install script file included in uploaded source - makes it 64 bit compatible, will overwrite existing Install script)

6. `sudo groupadd qtss`


7. `sudo useradd qtss -g qtss`

8. `cd DarwinStreamingSrvr6.0.3-Source; ./Buildit` to build - this will create the server binary in DarwinStreamingSrvr6.0.3-Source directory, along with some other binaries in some other subdirectories. "BuildIt" script comes with the download. Do not mind warnings. If build fails, you have to tweak this script. It is not that hard as it looks. Locate the line "case $PLAT" in the script. From that point on, it assigns some values to important compile/link variables depending on your $PLAT. $PLAT is a concatenation of `uname` and `uname -m` with a dot in between. Go ahead and find your $PLAT. Check if the Build script handles it. If you are using 32 bit linux, it should be "Linux.i686" or "Linux.i586". If you are 64 bit, it should be "Linux.x86_64". All these 3 cases are handled in the Build script - so you need to play around with the individual variables under that section if it is still not working. If your $PLAT is something else, and you think that the values from another case handler will work for you, go ahead and add a new case section and copy the lines from the one that works for you.

9. To Install: run `sudo ./Install` (You must be logged in as root to install Darwin Streaming Server)
- During installation, it will ask: "Please enter a new administrator user name:" type in some user name
- It will ask "Please enter a new administrator Password:" type in some admin password

10. After installation, DSS writes its binaries and files to various locations. Important directories are:
- /usr/local/sbin: server executable (DarwinStreamingServer) and admin server (streamingadminserver.pl)
- /usr/local/movies: default content directory (where streams are served from, already contains sample hinted streams)
- /usr/local/bin: some other binaries, including "StreamingLoadTool" a load test tool (discussed in the Performace/ Load Tests section)
- /etc/streaming: streamingserver.xml (configuration file for DSS), streamingloadtool.conf (configuration file for the load test tool StreamingLoadTool), qtusers and qtgroups file (contains users and groups for DSS, at this point only the admin user created during installation will be present) and
- /var/streaming/logs: DSS logs folder (StreamingServer.log, Error.log are the most important, also has streamingadminserver.log and mp3_access.log)

11. Run Server: cd /usr/local/sbin; sudo ./DarwinStreamingServer. 2 processes will run - the first one runs as root, and forks the main server as qtss. While doing a ps aux | grep Darwin, the one with the higher process id is the forked server process. If it crashes or gets killed, the other one immediately respawns another

12. Run Admin Server: sudo -u qtss /usr/local/sbin/streamingadminserver.pl. Target machine should have yapache installed and set up as a web server for accessing the administration console. If you are not allowed to impersonate qtss, just do a "sudo /usr/local/sbin/streamingadminserver.pl"

13. Access admin console from browser at: http://<your server name>:1220/parse_xml.cgi. Provide admin user id and password created during installation. Most of the parameters that you can change from the admin console write to /etc/streaming/streamingserver.xml

14. Play streams: Install apple's quick time player on any windows box that can resolve your linux machine's name, File->Open URL... and use rtsp://<your_server_name>/<any_sample_movie_from_/use/local/movies>. Note that DSS can only play "hinted" streams. Hinting can be done by Apple's tools, find out how to hint custom tracks if you want to play streams other than the sample ones

15. To stop server: kill the 2 server processes. If you only kill the one with higher pid, it will keep on getting re-spawned, so kill the lower one first. If you use -9, it results in the server not calling the cleanup modules, do not do that as a rule.

## Before you start writing your own modules to customize DSS... ##

This is the fun part for C++ developers. Before attempting to create your own module, you have to understand the architecture of the server. The architecture is pretty decoupled, as the entire server is already written with various modules strung together - many of the core server functionalities are implemented as modules.

The core server code is in the Server.tproj folder. After starting up, the server loads modules. Modules are either static or dynamic. Static modules are built when the server is built. When the server runs, it loads the dynamic modules first. It looks for shared libraries (unix so files, only that they are not named with so extension) under the directory /usr/local/sbin/StreamingServerModules. After that, it loads the static modules. The code for all modules that already come with the server are in APIModules directory.

If you just want to add custom functionality without tampering the actual server code, it is easiest to write and build a dynamic module - all you need to do is build the dynamic module as a shared object and place the binary in /usr/local/sbin/StreamingServerModules directory. Restarting the server will result in loading of the dynamic modules from this directory, including yours.

**Roles**

You must understand Roles in order to start creating your own modules.

Each module must register itself in one or more "role"-s. There are many roles: Register, Initialize, Filter, Route, RTSP Pre-Processing, RTSP Request, RTSP Post Processing, RTP Roles, RTSP Authentication Role, RTSP AUthorization role, Error Logging Role, etc. Think of roles as the building blocks in a flowchart. The Server first invokes the Register role. Followed by Initialize Role. Then it sits quietly, waiting for requests. Once it receives a RTSP Request, it invokes the Filter Role, and passes it the entire URI for "filtering" - modules subscribed to this Role have the privilege of getting a writable copy of the RTSP request: it can actually change any part of the request! The server then invokes the Route role - modules subscribed in this role gets a writable copy of the location (folder) from where to serve the content, and can effectively change the serving point (thereby re-routing the request to a new point). And so on.

The sequence in which the server invokes its roles is fixed (coded in the server). More than one modules can subscribe to a role and a module can subscribe to more than one roles. Some roles are exceptions - only one module can register for them. This "subscription" happens inside the code handler for the Register Role - all modules must register to this role. When called upon in this role, a module tells the server what other roles it is subscribing to.

**Pre-requisite - QTSSAPIDocs.pdf**

Read the QTSSAPIDocs.pdf file, however boring it might be. Though it is available in the Documentation directory, I have uploaded it along with the source zip just to be handy. It is the single most important document amongst all others in the Documentation directory. It is the bible for module developers. Unfortunately, it leaves many things to be desired. It is organized, but messily. Still, this article cannot substitute reading this document. This article is like a compass - it will guide you, give you a better bigger picture, but the nity gritty of code must refer to this pdf file. Come back and read the rest of this article again once you are done with QTSSAPIDocs.pdf.

## Let us create a custom Auth Module ##

Let us imagine a likely scenario: you want to allow/decline requests based on a logic that takes into account the connection url (which would be the case if you force your users to supply some specific signature as a query string) and client IP address (which would be the case if you want to restrict access to some subnetwork). You would need a custom Auth module.  

**Authorization/ Authentication in DSS**

How does DSS handle Authentication and Authorization? And how is Authorization different from Authentication?

I chose the example of a custom Auth module because this will bring us to the topic of how DSS handles authorization/ authentication. It does a fairly good job at whatever basic scheme it supports - but that is, at best, basic. It uses a rudimentary mechanism, and it might not suffice for your purpose. Let us review what DSS provides us with.

Right after installation, all users can access all movies from the movies directory. But say you want to set up n user accounts in such a way that each user will have access to his or her own content only. See http://dss.macosforge.org/ --> "Administrator’s Guide" to see how to add user accounts. Basically, it boils down to the fact that each user will have a subdirectory under the movies directory (/usr/local/movies - the main movies directory). Each such subdirectory will have a "qtaccess" file present, which will list the users that can access that given subdirectory. The master list of all users, along with their credentials are stored in /etc/streaming/qtusers file (password is stored in encrypted format), which only linux user qtss can access. 

Now, when a given user wants to play a file, he/she can send user id and password as part of the RTSP request. The process of validating that pair of user-id and password against the master list in qtusers is called Authentication. So the output of Authentication process is "yes this is a valid user id - password pair" or "no it is not". Simply passing Authentication does not guarantee that the user will be able to play the requested file, because the user may still not be listed in the qtaccess file of the subdirectory where the requested content lies. In other words, passing Authentication means "you are a valid user of the system". It does not mean "you have access to the requested file". Authorization is the other half, which is the process of allowing the user access to the requested content. Authorization may fail due to a variety of causes, including the scenario described above: the user is not listed in the qtaccess file. So bottomline: a user will pass Authentication if he/she supplies the correct password, but still will not be able to play the stream (more generically, access a resource) if he/she fails Authorization, which may happen due to a variety of causes.

Needless to say, the server calls "RTSP Autheticate Role" before "RTSP Authorize role".

Interestingly and correctly, DSS enforces that only one module can subscribe for "Authenticate" Role, but unlimited number of modules can subscribe for "Authorize" role. That corroborates our conclusion above: Authentication will fail only under one circumstance: if you are an unknown user. But Authorization may fail under various situations, and there may be multiple modules, each verifying some aspect of Authorization. Thus, when the server calls the module subscribed for "Authenticate" role, it expects that module to validate the user id/ password. If that passes, it then invokes all the modules signed up for "Authorize" role - and each one of them can then apply custom rules to either approve or deny that RTSP request. For example, an Authorize module may deny the request because of too high bandwidth or too many number of open connections. Such custom authorization must be implemented as separate modules subcscribing to the Authorize role.

**Implementing a completely different auth scheme**

This mechanism of validating users (looking for them in qtusers file and so on) is implemented as a static module (you guessed it - that's the good thing guys - core features of DSS are simply modules). It is QTSSAccessModule. Go ahead, look at its code from the APIModules directory, getting your feet wet is always a good thing.

If you see the code of QTSSAccessModule, you will see that it has registered to the Authenticate Role (look in QTSSAccessModuleDispatch method). Of course, in addition to that, it is also regsiered to 5 other roles, including the Authorize Role. But the fact that it is registered to Authenticate role is significant, because Authenticate Role is one of the roles which only one single module can register to. So if you want to register another new module to Authenticate role, it's not gonna work!

So if we were to married to the idea that our new module must register to Authenticate role, we would be forced to retire the QTSSAccessModule. That may be okay, but it is not safe to write off the core modules of DSS just like that. Who knows what else that module must be doing? Anyway, QTSSAccessModule is a static module, it is not dynamic - so getting rid of it means getting the hands dirty with server code. Alternatively, we can edit QTSSAccessModule and write our code inside that module. Again, something that is not elegant.

For the kind of problem we are trying to solve, Authorize Role fits the bill perfectly.

One way to suppress whatever QTSSAccessModule does is not to have any qtaccess file anywhere. Which means all users, including anonymous users (those who do not supply any username or credentials while submitting the RTSP request), are allowed access to all content. QTSSAccessModule will pass all requests. Now we can add a custom module implementing Authorize role, and add the custom logic to allow/ deny in there. This is a simplistic approach indeed, designed to solve our fabricated needs. This is to just give you an idea - your situation will surely warrant a slightly different solution.

**Writing the code**

A word on your reference copy: the entire code for QTSSMyAuthModule, our glorious first DSS module, is inside the source zip: as a directory that you can just copy to APIModules folder, alongside existing modules.

**Building an existing module as dynamic library**

Before starting to write the code for a new module, you should learn how to build (make) an existing module - how to build only a module (not the whole server) as a shared library so that you can simply drop the module binary into the /usr/local/sbin/StreamingServerModules directory.

With some tweaking of the build scripts, I came up with one that I could copy directly to the subdirectory containing the code for the module. Each subdirectory under APIModules directory already contains a Makefile.POSIX - but that cannot be used directly, as it expects some exported variables that the global build script (remember "Buildit"?) populates before invoking it. But to build a module separately, we need a local build script that will enable us to similarly use the local Makefile.POSIX. On top of that, I tweaked it some more to output a shared library, not a static one. The result is "build" - refer source zip/QTSSMyAuthModule.

Try it on an existing module first, say QTSSHomeDirectoryModule. Simply copy the "build" script I have uploaded to QTSSHomeDirectoryModule folder. Edit the script's final echo statement (also you might need to run chmod to make it executable) to print out the correct module's name, and you should be done. Run "build clean" first and then run "build", and see the object files and the binary called "QTSSHomeDirectoryModule" getting created.

Now copy that binary to /usr/local/sbin/StreamingServerModules. Restart the server. Look at /var/streaming/logs/Error.log at this point, and it should have freshly logged a list of modules the server just loaded. For each module, it specifies whether it is static or dynamic. If you are successful in building QTSSHomeDirectoryModule, it should have it listed as loaded and dynamic. Repeating the experiment by removing the binary from /usr/local/sbin/StreamingServerModules will result in that line of log not appearing.

**Time to create QTSSMyAuthModule**

Copy any existing module (The following tutorial assumes you have used QTSSHomeDirectoryModule) source code in a new subdirectory "QTSSMyAuthModule" and change all references to the old name to new name.

There are 2 roles that QTSS Home Directory Module handles which we do not need to: RTSPRoute Role and ClientSessionClosing Role. I am not going to explain all these roles here, please read the bible for that. Remove the code handling those roles. The other 4 (Register, Initialize, RereadPrefs and RTSPAuthorize) we need. And we need an extra role that is not in there: Shutdown Role, so add code for that one (see uploaded source zip).

Let me give you a quick idea of the roles we will handle in QTSSMyAuthModule.

**Register, Initialize and Shutdown**: Register is mandatory (this is when you tell the server what other role you plan to register to). Initialize is a good idea to have - e.g., I added a new class in a separate H - CPP file pair, and I needed a static object of my class type. So I created an instance in Intialize (by calling new() or QTSS_New()). Naturally question of cleaning up this memory arises. In such a case where we need to call cleanup code at the end, we must subscribe to another role: Shutdown Role, so that our module is called when the server is shutting down.

**RereadPrefs**: RereadPrefs is also good to have especially if you want your new modules to be controlled by configuration settings. Adding configuration settings is very easy: just edit /etc/streaming/streamingserver.xml and add, under <CONFIGURATION>, a new node: <MODULE NAME="<Your module name, in this case QTSSMyAuthModule>> with as many configuration parameters as needed. For example, I wanted a master setting which enabled/disabled by auth plugin, so I added <PREF NAME="enabled" TYPE="Bool16">true</PREF>. Once the setting is added, the handler for RereadPrefs Role should simply read in the value of the setting using QTSSModuleUtils::GetAttribute(...) (see my code) Having the module subscribe to RereadPrefs Role simply ensures that your module can read in changed settings without a server restart.

**RTSPAuthorize**: For RTSPAuthorize handler, I modeled my code after how QTSSHomeDirectoryModule handles the Authorize Role. I noticed that the server calls my module several times for playing back a single stream - and I only need to authorize once (the first time) to be efficient. On all subsequent calls, I have to figure out that this rtsp session is already authorized. A custom attribute added to the RTSPSession object helped me to do that, and the technique is clearly implemented in the existing code of QTSSHomeDirectory code. As part of my implementation of Authorize role, I had to extract the RTSP URL and Client IP for the request. Combing through the code of existing modules, I found URL extraction technique from QTSSSplitterModule.cpp (see QTSSReflectorModule subdirectory) - but the string extracted there did not start from "rtsp://" but only from the filename part. Scanning QTSSAPIDocs.pdf, I found lists of all supported attributes for the standard objects, and located one that contains the URL in the format I need.

Thus, as you see, no article can show you the exact technique you need to know - there is an ocean of information among the code of existing modules and QTSSAPIDocs.pdf, though scattered and difficult to locate. How quickly you find the technique you need from the existing code base and/or the documentation depends how much back you broke.

**Using your own classes**

I will refrain from explaining each line of my code of QTSSMyAuthModule - the place where the exact authentication is performed is full of running comments. I separated the functionality of allowing/denying to a separate class - MyAuth.cpp, MyAuth.h.

I have seen that I could not use STL classes and methods directly into QTSSMyAuthModule.cpp (the main template file that houses the mandatory methods a module must have) - they conflict with some header file DSS uses. That should be fixable with some effort, but I opted for the easier route: if I follow my scheme of including a separate class, I can use STL in MyAuth.h/cpp without any problem.

**How configuration works**

/etc/streaming/streamingserver.xml is precise and clear. It has all the server's settings listed under /CONFIGURATION/SERVER, followed by the settings for all the modules, each with node /CONFIGURATION/MODULE@NAME . Changing these settings do not need a server restart - only sending the HUP signal to the server process (one with the higher pid). For a custom dynamic module to re-load preferences at the same time when the server does (in response to HUP signal), it must implement the RereadPrefs Role.

**How logging works**

**Error Logging**

Though DSS has all modules in the APIModules directory, I found one that is hidden in Server.tproj folder - QTSSErrorLogModule. That is probably because this module is deemed an integral part of the server itself, and becasue it is so important. It is because of this module that any other module and the sever itself can write to the Error.log file located in /var/streaming/logs.

Even informational logs can be simply written to the Error.log file with verbosity parameter = qtssDebugVerbosity, and it won't actually show up unless /CONFIGURATION/SERVER/error_logfile_verbosity is set to 4 (qtssDebugVerbosity). The possible values of this configuration setting is: 0 (Fatal Verbosity), 1 (Warning Verbosity), 2 (Message or INFO Verbosity), 3 (Assert Verbosity) and 4 (Debug). Whatever the value is set to, logs written with verbosity parameter equal to or less than that value will actually get written. Writing a log with Fatal Verbosity causes the server to shut down.

In order to write to the error log stream from anywhere in a custom module, we need to save the error log stream in a static QTSS_StreamRef object in the Initialize handler method, where the parameter passed to the handler has a reference to the stream. Please refer to my code.

**Stream Request Logging**

The module QTSSAccessLogModule does this for us: it logs a bunch of statistics for every rtsp session closing/ playback stopping (to /var/streaming/logs/StreamingServer.log). Unlike flash media server, it does not trap pause/unpause. Any change/addition to the exact fields being logged should be easily achievable by tweaking the code of this module. The streamingserver.xml file has all the configurable parameters for this module in the corresponding MODULE node.

**Server monitoring/ Heath check support**

DSS has a cool feature: turn on the entry "enable_monitor_stats_file" in streamingserver.xml, and it will output a stats file to the same folder as Error.log is written to (which is /var/streaming/logs, but you can change it using the "error_logfile_dir" entry in streamingserver.xml). This stats file is refreshed every X seconds, where X is "monitor_stats_file_interval_seconds".

Worth mentioning here is my encounters with this monitoring feature: the file has XML format, so I decided to monitor my server using a browser. But somehow, DSS writes some gibberish to the top part of the stats file, which makes the XML malformed, so the browser cannot render it. Also, the permissions of the file is set such that not everybody can read the file. I also wanted to output the file anywhere I wanted (using the "monitor_stats_file_name" entry), but DSS is written to expect only the file name from that entry, not the entire path. My adventures to fix this led me to Server.tproj/RunServer.cpp, LogStatus() function, where I could fix all these. Now I output my stats file to /tmp, and my apache server serves the file fine, because I have created a sym link to the real file from apache's htdocs directory. And it is pure XML now, no gibberish.

## Performance and Load tests ##

I am not happy with the load tests I ran on DSS. It cannot take thousands of concurrent users. Someday I might take the time and effort to try to fix (read improve) that in the code, but let me first point you to how I ran these tests.

**StreamingLoadTool**

DSS comes with a load test tool, /usr/local/bin/StreamingLoadTool. Pass -v to it to know about the other parameters it takes. I used the -f <configuration file> parameter mostly, and used the config file to specify everything else. The sample config file is located as /etc/streaming/streamingloadtool.conf.

I could successfully run load tests from a different client machine - all it takes is to copy the load test binary and the configuration file over.

I found that the StreamingLoadTool that comes with DSS 6.0.3 crashes when I run it with runforever=yes and concurrent number of clients=100. Pfft.

**openRTSP**

A better tool to load test DSS is openRTSP from www.live555.com. If you want to build it from source, you have to download the entire live555 streaming media source code. I went ahead and followed their instruction, and it went smoothly. You will end up with a "live" directory at the end - it's your choice where to put it, but that contains everything. openRTSP, the command line RTSP client tool, is inside testProgs folder. It has excellent documentation, so I will refrain from explaining how it should be used. I wrote a script that spawned lots of openRTSP sessions, each with -c option (for continuous play), -r (do not render video) and -p 80 (to redirect the video to that port), and I controlled the number and rate of spawning.

**Results**

The result is that DSS, with all optimizatoins I could think of (increase the maximum number of open file handles in your system to a gigantic number using /etc/security/limits.conf, set "maximum_bandwidth" and "maximum_connection" in streamingserver.xml to -1, "run_num_threads" to various values, including 0 for auto-detection of number of cores) - DSS cannot handle more than 350 concurrent users.

```
# of concurrent users	CPU (Forked Server)	CPU Idle (Overall)	Memory (Server Process)	Stable?	Bandwidth (MBPS)
100	1%	99%	4 MB	yes	3.75
200	2%	97%	7 MB	yes	7.5
300	3%	96%	12 MB	yes	11.25
350	4%	96%	21 MB	yes	13.125
400	4%	96%	22 MB	no	 - 
```

The bandwidth numbers above is a straightforward calculation considering I was streaming 300 kbps files. DSS sample files also comes with 1000 kbps files and high definition files, I am sure these files will result in much higher bandwidth, but performance and latency using the high bitrate files was not measured in my experiments - I will await feedback from readers on how that went.

I did not notice any memory leaks - in stable mode, it ran constantly for weeks, processing 350 continuous and concurrent users, with no leaks perceived.

**Running multiple DSS servers for scalability**

For some, that could be enough. I needed more. There is a solution though - even when DSS topples over with 400 users, it uses virtually no CPU & memory (which indicates that the bottleneck is somewhere in the code, and that makes me bolder when I think that it can be fixed). This "leanness" can be effectively used by running multiple DSS servers!

While executing the server, we can pass -c <whatever streamingserver.xml we want>. If we create multiple streamingserver.xml files, each with a different port listed in "rtsp_port" entry (like 7070 in one, 554 in another, and so on), and then execute multiple DSS instances each using a separate streamingserver.xml as configuration file, we can achieve this. I have parallelly run 3 instances of the server, each processing 350 concurrent users - that is one way to reach four digits - 1000 users!

Only caveat is the url you pass to openRTSP (or whatever client you are using) must access the correct instance of the server using the port number specified using a colon after the hostname. If you are thinking of using this setup in real world, you have to write another application - a software load balancer, that will listen on any given port, and delegate the request to any of the N running DSS instances. Clients then must connect to that port always. I have not yet tried this thing out though.

## RTSP, RTP, UDP, RUDP, TCP... ##

While dealing with Darwin Streaming Server, you will come across these acronyms. At times you will have to choose one over another. For example, while setting up StreamingLoadTool (the load test tool that comes inherently with DSS), its configuration file has entries where you pick the mode of execution - RUDP? UDP? TCP? You can retain the defaults if you are not sure what to pick.

Explaining the differences and meanings of these is beyond the scope of this article. May be in a future article. I mention these here because you need to understand these concepts to get the most out of DSS. If you are only setting up a server for your users to use - you may ask - why do you care whether it is doing RUDP or UDP? At the beginning, you might not need to care. But if lots of users are using your service, you may be asked which one gives higher throughput. They may be using clients of varying capabilities, and ones that gives it users the option to pick a mode.

For absolute beginners, just inderstand this much: RTSP is the protocol of delivery (corresponds to HTTP). RTP is the underlying protocol used for packets transfers by RTSP. TCP and UDP are alternate network protocols - TCP is characterized by high reliability but low timeliness, as it involves a lot of handshaking (ACK-s, NACK-s etc). UDP is characterized by low reliability (packets may not make it to the other end) but high timeliness. R-UDP is Apple's proprietary version of UDP that adds some handshaking, thus they call it "Reliable UDP". Here is an article that attempts a better explanation: http://www.vbrick.net/Topics/RTSP.asp.

You may ask why does DSS need to deal with these concepts? Any streaming server would, as its sole job is to deliver a streaming media to the client - which, as opposed to playing back a file, is characterized by the fact that nothing is stored on the client side. Bits must be transmitted exactly at the speed that the video being played demands - a lighter scene may need 100,000 bits per second, and a darker scene may need 700,000 bits per second. This number is called the bitrate of the video. Coordinating this transfer, where the speed goes up and down, ensuring reliability and handshaking is the complex domain of streaming servers. And it cannot be solved by HTTP, as HTTP can only fetch the whole file (it can also fetch partial files based on byte range requests) but it cannot fetch a file continuously over the same connection. RTSP, RTMP, etc are protocols used for this purpose.

## Conclusion ##

Apple guys have coded this well. A reviewer, on afterthought, can always point out areas of improvement, but the project is so massive that finally ending up with a coherent, working and mostly bugfree set of files of this magnitude is a commendable achievement.

Usage of so many static variables is something I do not like - it exerts pressure on the heap. And amateurishly conceived utility classes like StrPtrLen literally drives me mad - these guys have created a class to hold a pointer to sombody else's data! Please be extremely careful while using this almost criminal class - it invites misuse. Overall, the code could have been neater, but hey, I did not pay for this. I will take what DSS is worth for.


## License ##

This article, along with any associated source code and files, is licensed under The Code Project Open License (CPOL)
