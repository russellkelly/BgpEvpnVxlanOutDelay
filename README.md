# BgpEvpnCoreMonitor
An Arista EOS SDK Agent that Monitors the Status of BGP EVPN peers, and enables/disables ESI interfaces depending on the status of the said BGP EVPN Peers.  The Figure below details this behaviour.

<img src="BgpEvpnCoreMonitor-Overview.jpg" alt="Drawing"  height="300" width="750">
               Figure 1: Overview of the failure case the BgpEvpnCoreMonitor agent addresses


## Installation
BgpEvpnCoreMonitor may be installed using the RPM provided or manually.

The RPM is installed using the usual steps for an EOS extension (survivable over reboots):

Copy the RPM to the switch

```
scp BgpEvpnCoreMonitor-<version>.rpm <user>@<switch-ip>:/mnt/flash/BgpEvpnCoreMonitor-<version>.rpm
```

Now install the RPM as an extension

```
Arista#copy flash:BgpEvpnCoreMonitor-<version>.rpm extension:
Copy completed successfully.

Arista#extension BgpEvpnCoreMonitor-<version>.rpm 
```
To verify that the extension is installed successfully: 

```Arista#show extensions```

The RPM should have automatically configured the BgpEvpnCoreMonitor daemon as below:

```
daemon BgpEvpnCoreMonitor
exec /usr/local/bin/BgpEvpnCoreMonitor
no shut
```
If you want to set the switch so that the agent only checks for ESI interfaces once upon initialization, set the option ESICHECK to "False".  This can be safely used if the number of ESI interfaces configured does not change.   

The benefit of skipping this change is that the interfaces can be enabled/disabled a little faster, as the sub routine to check for configured ESI interfaces at time of BGP EVPN failure is skipped.

```
daemon BgpEvpnCoreMonitor
exec /usr/local/bin/BgpEvpnCoreMonitor
option ESICHECK value False
no shut
```

The default is True, whereby whenever all BGP EVPN peerings are newly UP or DOWN, the active ESI interfaces are checked before being enabled or disabled respectively.

Lastly, if it is desired that BgpEvpnCoreMonitor be loaded automatically at boot time, the boot extensions must be modified accordingly:

```
Arista# show installed-extensions
BgpEvpnCoreMonitor-<version>.rpm 

Arista# show boot-extensions

Arista# copy installed-extensions boot-extensions
Copy completed successfully.

Arista# show boot-extensions
BgpEvpnCoreMonitor-<version>.rpm 
```


Alternatively, BgpEvpnCoreMonitor may be installed manually.  

```NOTE: This installation method does not survive router reboots!```

The first step in doing so is to copy the BgpEvpnCoreMonitor script to the switch:

```
scp BgpEvpnCoreMonitor <user name>@<switch IP address>:/mnt/flash
```
Next, BgpEvpnCoreMonitor must be configured to interact with Sysdb by dropping into bash on the switch and executing	

```
sudo cp /usr/lib/SysdbMountProfiles/EosSdkAll /usr/lib/SysdbMountProfiles/BgpEvpnCoreMonitor
```
And then editing ```/usr/lib/SysdbMountProfiles/BgpEvpnCoreMonitor```, changing first line to be:

```agentName:BgpEvpnCoreMonitor-%sliceId```

Lastly, the BgpEvpnCoreMonitor daemon may be started using the conventional EOS daemon CLI:

```
daemon BgpEvpnCoreMonitor
exec /usr/local/bin/BgpEvpnCoreMonitor
no shut
```
