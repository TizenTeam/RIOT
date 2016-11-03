IoTivity Examples
===============
These examples implement some simple clients and server to test the IoTivity package for RIOT-OS. The pkg is based on the [IoTivity-Constrained][1] library.
All examples use the realm-local multicast address **ff03:158** instead the link-local multicast address **ff02::158**. Every payload is [CBOR][2] encoded.

##Server Example
This example implements an IoTivity Server that contains 4 resources. 
 1. */oic/res*: returns the representation of resources available on the server.  Defined [here][3].
 2. */oic/p*: returns information about the platform on which the server runs. Defined [here][3].
 3. */oic/d*: returns information about the device on which the server runs. Defined [here][3].
 4. */light/1* : it is a light (LED) management resource (implemented in resources/res-light.c). Resource Type: *oic.r.light*.
	- GET: returns the status of the LED. The payload is: `{state: true/false}`
	- PUT: changes the status of the LED. The payload is: `{state: true/false}`
	- OBSERVE: the registered client is notified when the status changes. The payload is the same of the GET response.

----------

## Client Example
This example implements a simple client. It is able to discover resources with ResourceType *oic.r.light*. Once it finds a resource, it registers for OBSERVE notifications, then it changes the status every second by sending a periodic PUT request. 


----------

##Client_Switch Example
This example implements a simple client. It is able to discover resources with ResourceType *oic.r.light*. Once it finds a resource, it registers for OBSERVE notifications. It changes the status when the User Button is pressed. If the button  is not present it is just an observer.


----------

##BR_FW Example
It is an "enhanced" version of the GNRC Border Router. It implements a simple forwarder (UDP server/client) for multicast requests with destination ff03::158 port 5683.

[1]: https://github.com/iotivity/iotivity-constrained/
[2]: http://cbor.io/
[3]: https://openconnectivity.org/resources/specifications
