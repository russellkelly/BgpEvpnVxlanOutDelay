# BgpEvpnVxlanOutDelay

The purpose of this agent is to monitor MLAG reload timers, along with MLAG enablement in general, and disable all BGP EVPN peers if the reload timers are active.  Thus effectively simulating BGP out delay.

When the MLAG reload timers expire the BGP peers are re-enabled.  This allows for all MLAG and MLAG/non-MLAG links to stabilize before bringing the BGP EVPN peers up.  Note. IPv4 BGP peers are up, so connectivity can be verified.

These enablement/disablement changes are triggered via inotify, which is monitoring the logfile on the router for any BGP Agent and MLAG agent changes.


```
daemon BgpEvpnVxlanOutDelay 
   exec /usr/local/bin/BgpEvpnVxlanOutDelay
   no shut
   
 ```
This requires the EOS SDK extension installed if its < EOS 4.17.0 release.
All new EOS releases include the SDK.


## Installation
BgpEvpnVxlanOutDelay may be installed using the RPM provided or manually.

The RPM is installed using the usual steps for an EOS extension (survivable over reboots):

Copy the RPM to the switch

```
scp BgpEvpnVxlanOutDelay-<version>.rpm <user>@<switch-ip>:/mnt/flash/BgpEvpnVxlanOutDelay-<version>.rpm
```

Now install the RPM as an extension

```
Arista#copy flash:BgpEvpnVxlanOutDelay-<version>.rpm extension:
Copy completed successfully.

Arista#extension BgpEvpnVxlanOutDelay-<version>.rpm 
```
To verify that the extension is installed successfully: 

```Arista#show extensions```

The RPM should have automatically configured the BgpEvpnVxlanOutDelay daemon as below:

```
daemon BgpEvpnVxlanOutDelay
exec /usr/local/bin/BgpEvpnCoreMonitor
no shut
```

Lastly, if it is desired that BgpEvpnVxlanOutDelay be loaded automatically at boot time, the boot extensions must be modified accordingly:

```
Arista# show installed-extensions
BgpEvpnVxlanOutDelay-<version>.rpm 

Arista# show boot-extensions

Arista# copy installed-extensions boot-extensions
Copy completed successfully.

Arista# show boot-extensions
BgpEvpnVxlanOutDelay-<version>.rpm 
```


Alternatively, BgpEvpnVxlanOutDelay may be installed manually.  

```NOTE: This installation method does not survive router reboots!```

The first step in doing so is to copy the BgpEvpnVxlanOutDelay script to the switch:

```
scp BgpEvpnVxlanOutDelay <user name>@<switch IP address>:/mnt/flash
```
Next, BgpEvpnVxlanOutDelay must be configured to interact with Sysdb by dropping into bash on the switch and executing	

```
sudo cp /usr/lib/SysdbMountProfiles/EosSdkAll /usr/lib/SysdbMountProfiles/BgpEvpnVxlanOutDelay
```
And then editing ```/usr/lib/SysdbMountProfiles/BgpEvpnVxlanOutDelay```, changing first line to be:

```agentName:BgpEvpnVxlanOutDelay-%sliceId```

Lastly, the BgpEvpnVxlanOutDelay daemon may be started using the conventional EOS daemon CLI:

```
daemon BgpEvpnVxlanOutDelay
exec /mnt/flash/BgpEvpnVxlanOutDelay
no shut
```
