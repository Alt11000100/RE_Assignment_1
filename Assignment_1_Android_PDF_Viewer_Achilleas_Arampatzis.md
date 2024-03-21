
### Assignment 1 Android

We will first use apktool to .xapk. In the manifest.json we see that the apk is split into few apks but the main is com.tragisoap... . Already we can see that there are some "strange" permissions. With the help of jadx-gui we decompile the base .apk "com.tragisoap.fileandpdfmanager.apk".

In the AndroidManifest.xml we can see some suspicious permissions:
![[[20240319155728.png](https://github.com/WokeToThis/RE_Assignment_1/blob/main/Pasted%20image%2020240319155728.png)]] 

So the app can potentially install malicious packages.
Assuming that the app will connect to a server to install the malicious packages we search for "http". Indeed we find 2 strings:

![[Pasted image 20240319160302.png]]

Searching virustotal for these domains we see that they are related to malware. 

Before going further we need to check what are the entry points of the application.
![[Pasted image 20240319173340.png]]
this is asking the user to accept access to external storage permission.

### Cortina

![[Pasted image 20240319233431.png]]
After the connection with the server and having adjusted all permissions to download more files, the malicious application opens a url connection to download the main malicious 1.apk:
![[Pasted image 20240319233622.png]]
what is interesting is that there are many checks before:
![[Pasted image 20240319233952.png]]
![[Pasted image 20240319234009.png]]
This suggests that the malware will check to see if it is in a simulated environment and not run. If, however the brand is samsung or moto(?) it will run. Also, before downloading the last apk, it will check for the country and not run on specific ones:
![[Pasted image 20240319234233.png]]
Depending the country will also provide a message about "updating" ![[Pasted image 20240319234431.png]]
while downloading the malicious code. The app will ask the user to accept trusting downloading from untrusted sources:
![[Pasted image 20240319234735.png]]

After some troubles with the 1.apk I managed to read some of the AndroidManifest by importing the 1.apk in cybershef, taking the hexdump and removing the null bytes:
![[Pasted image 20240321014639.png]]
as it seems the app has some more permissions that should not have.
![[Pasted image 20240321014727.png]]
also it gets the bind_accessibility_service which will give to the malware viewing control of everything that is happening on the device.

Going into the classes.dex we see that obfuscation is used:
![[Pasted image 20240321181510.png]]

The malicious app uses xor to produce the real string ![[Pasted image 20240321181552.png]]
but I was not able to reconstruct the real string from that code.

At this point I could not find anything else that is substanstial. It was interesting for me that it had some alibaba and alipay folders, but I could not find the exact purpose of the code. Because of the permissions I assume that maybe the obfuscated part is capable of downloading more packages with malicious code. 

### Dynamic analysis 

I lost a lot of time trying to uncompress correctly the 1.apk to read the AndroidManifest.xml. I assumed it had something to do with the header, but I was not able to correct it. 
Installing the 1.apk we see that it asks for permissions:
![[Pasted image 20240321204825.png]]
![[Pasted image 20240321204907.png]]
This could potentially affect the user with a keylogger and full access control to the device.
But we get an error "unsupported device" because we didn't patch the application and it detects that we use an emulator.
My next steps would be to patch the application so that it doesn't check for emulator and intercepting the communications with the servers. I assume that it would download another file that would have the real malicious code that would be able to put a keylogger on the device, read and send sms, turn off battery optimisations so that it can run in the background all the time and probably delete the previous files and install new packages.

###  Summary 

The malicious app runs at 4 stages: Firstly the user has to download it and give permission to install more packages. When this is done the app communicates with some servers to download cortina,muchaspuchas and 1.apk. With the help of these files 1.apk can run and ask for more permissions like Accessibility services and foreground permission to work on the background. When these permissions are granted, the app has effectively full viewing control over the device of the user and can install keyloggers, send and receive sms , steal user credentials and biometrics.  Some interesting details are that it will only affect certain countries, it uses obfuscation at certain levels and some checks to catch if it is going to run on an emulator.


