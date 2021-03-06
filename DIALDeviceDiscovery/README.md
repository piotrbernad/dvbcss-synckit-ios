# <small>dvbcss-synckit-ios</small><br/>DIALDeviceDiscovery Framework


* **[Overview](#overview)**
* **[How to use](#how-to-use)**
[](---START EXCLUDE FROM DOC BUILD---)
* **[Read the documentation](#read-the-documentation)**
[](---END EXCLUDE FROM DOC BUILD---)
* **[Run the example app](#run-the-example-app)**

This framework provides APIs for the discovery of services via SSDP and the discovery of devices via DIAL, as used in [HbbTV 2](http://hbbtv.org/resource-library/#specifications) compliant TVs It allows HbbTV terminals existing on the same network to be discovered by the Companion Screen Application.
Using the DIAL protocol, the Companion Screen Application can launch an HbbTV application on the television.

#### Dependencies
This framework requires the following dynamic libraries (iOS frameworks) (found in SyncKit):

  * SyncKitConfiguration
  * SyncKitCollections
  * SimpleLogger
  * AsyncSocket

[](---START EXCLUDE FROM DOC BUILD---)
## Read the documentation
The docs for this framework can be read [here](https://bbc.github.io/dvbcss-synckit-ios/latest/DIALDeviceDiscovery/).
[](---END EXCLUDE FROM DOC BUILD---)


## Overview

#### Service Discovery via SSDP

The [SSDPServiceDiscovery](https://bbc.github.io/dvbcss-synckit-ios/latest/DIALDeviceDiscovery/Classes/SSDPServiceDiscovery.html) class implements the service discovery API of the library. The API aims at discovering UPnP devices supporting a particular service type in the local network. This class listens for service advertisements on the UPnP multicast address/port and, maintains soft-state. Changes in services table membership are notified to a delegate via the SSDPServiceDiscoveryDelegate protocol.


#### Device Discovery via DIAL

The [DIALServiceDiscovery](https://bbc.github.io/dvbcss-synckit-ios/latest/DIALDeviceDiscovery/Classes/DIALServiceDiscovery.html) class implements the API to discover and manage devices on the network running a DIAL server and applications e.g. an HbbTV application.
In the first step of the discovery process, it uses the SSDPServiceDiscovery module to discover services of type *DIAL_MultiScreenOrgService1*. Subsequently, [DIALAppDiscoveryTasks](https://bbc.github.io/dvbcss-synckit-ios/latest/DIALDeviceDiscovery/Classes/DIALDeviceDiscoveryTask.html) are created and launched to take over the remainder of the device discovery: 1) the Device Description request and 2) the Application Information request.

The discovery of new devices on the network or expiry of currently-known devices by the library is notified to observers via the DIALDeviceDiscoveryDelegate protocol and via event notifications.


## How to use

#### 1. Add the framework and its dependencies to your project

A Cocoa Pod version of this library is not currently available. To use this library, you must add it and its dependencies to your project.

You will need the following projects from this repository:

  * [DIALDeviceDiscovery](.)
  * [SyncKitConfiguration](../SyncKitConfiguration)
  * [SyncKitCollections](../SyncKitCollections)
  * [SimpleLogger](../SimpleLogger)
  * [AsyncSocket](../AsyncSocket)

Either include them in your workspace and build them; or build them in this repository's [synckit.xcworkspace](../synckit.xcworkspace)

Then add them under *Linked Frameworks and libraries* in your project's target configuration:

  * DIALDeviceDiscovery.framework
  * SyncKitConfiguration.framework
  * SyncKitCollections.framework
  * SimpleLogger.framework
  * AsyncSocket.framework

If you have two XCode instances running you can drag and drop the framework files from their respective project in SyncKit to your own project.


#### 2. Import the header files

Import this header file into your source file:

```
#import <DIALDeviceDiscovery/DIALDeviceDiscovery.h>

```
#### Initialise the DIAL discovery engine and register for notifications
Create a DIALDeviceDiscovery instance to look for DIAL supporting devices with an "HbbTV" DIAL application:

```
config = [SyncKitGlobals getInstance];
assert(config!=nil);

LogInfo(@"AppDelegate: Loading DIAL device discovery components .... ");

self.tvDiscoveryComponent = [[DIALServiceDiscovery alloc] initWithApplication:@"HbbTV"];
[self.tvDiscoveryComponent start];
```
Register for notifications:

```
// register for notifications
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(DIALServiceDiscoveryNotificationReceived:) name:nil object:self.tvDiscoveryComponent];

```

#### Get all currently discovered devices
Call the getDiscoveredDIALDevices() method to get a list of all currently available devices. Each device is represented by a [DIALDevice](https://bbc.github.io/dvbcss-synckit-ios/latest/DIALDeviceDiscovery/Classes/DIALDevice.html) object

```
NSArray*  deviceList =  [_tvDiscoveryComponent getDiscoveredDIALDevices];;

```

#### Wait for and handle notifications about new devices discovered and other devices leaving

Handle kNewDIALDeviceDiscoveryNotification  and kDIALDeviceExpiryNotification notifications.
Here's an example of an event handler:

```
/** Handler for DIAL service discovery and update notifications */
- (void) DIALServiceDiscoveryNotificationReceived:(NSNotification*) aNotification
{
    DIALDevice *device;

    if ([[aNotification name] caseInsensitiveCompare:kNewDIALDeviceDiscoveryNotification] == 0) {
    id temp = [[aNotification userInfo] objectForKey:kNewDIALDeviceDiscoveryNotification];

    if ([temp isKindOfClass: [DIALDevice class]] )
    {
      device = (DIALDevice*) temp;

      MWLogDebug(@"App Delegate: Master Device %@ found on network.", device.friendlyName);

      // do something ...
    }


    } else if ([[aNotification name] caseInsensitiveCompare:kDIALDeviceExpiryNotification] == 0)
    {
      device = (DIALDevice*) [[aNotification userInfo] objectForKey:kDIALDeviceExpiryNotification];

      LogInfo(@"App Delegate: Master Device %@ no longer available.", device.friendlyName);

      // do something e.g. tell observers to refresh their device list

      [[NSNotificationCenter defaultCenter] postNotificationName:kRefreshDiscoveredDevicesNotification object:nil];

    // ...

    }
}

```

## Run the example app
A demo app called [DIALDiscoveryDemo](DIALDiscoveryDemo/) is included with the  library. To run the demo,open the DIALDiscoveryDemo workspace (DIALDiscoveryDemo.xcworkspace) in XCode and deploy on your iOS device.
Make sure your TV and your iOS device are both on the same network.

If you want to run your own DIAL server. There are other projects that implement this, such as node_hbbtv by Fraunhofer. The latest build 0.0.10 or later is required. Install it on a computer (on the same network) from npm or obtain it directly from github:
https://github.com/fraunhoferfokus/node-hbbtv/


## Licence

This framework is developed by BBC R&D and distributed under the Apache License, [Version 2.0](http://www.apache.org/licenses/LICENSE-2.0).

© Copyright 2016 BBC R&D. All Rights Reserved
