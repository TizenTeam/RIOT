IoTivity Examples
===============
These examples implement some simple clients and server to test the IoTivity package for RIOT-OS. The pkg is based on the [IoTivity-Constrained][1] library.
All examples use the realm-local multicast address **ff03:158** instead the link-local multicast address **ff02::158**. Every payload is [CBOR][2] encoded.
All examples have been tested on Native and SAMR21-XPRO.

##Server Example
This example implements an IoTivity Server that contains 4 resources. 
 1. */oic/res*: returns the representation of resources available on the server.  Defined [here][3].
 2. */oic/p*: returns information about the platform on which the server runs. Defined [here][3].
 3. */oic/d*: returns information about the device on which the server runs. Defined [here][3].
 4. */light/1* : it is a light (LED) management resource (implemented in resources/res-light.c). Resource Type: *oic.r.light*.
	- GET: returns the status of the LED. The payload is: `{state: true/false}`
	- PUT: changes the status of the LED. The payload is: `{state: true/false}`
	- OBSERVE: the registered client is notified when the status changes. The payload is the same of the GET response.

## Client Example
This example implements a simple client. It is able to discover resources with ResourceType *oic.r.light*. Once it finds a resource, it registers for OBSERVE notifications, then it changes the status every second by sending a periodic PUT request. 

##Client_Switch Example
This example implements a simple client. It is able to discover resources with ResourceType *oic.r.light*. Once it finds a resource, it registers for OBSERVE notifications. It changes the status when the User Button is pressed. If the button  is not present it is just an observer.

##BR_FW Example
It is an "enhanced" version of the GNRC Border Router. It implements a simple forwarder (UDP server/client) for multicast requests with destination ff03::158 port 5683. 

#Scenarios
It is possible to deploy 2 different scenarios with these examples. 
##Node-to-Node Communications
In this scenario, we will deploy an IoTivity Client and IoTivity Server on different nodes. We can choose two different clients for this scenario: client (periodic PUT) or client_switch (PUT sent on Button pressed). The first one runs well both on native either on SAMR21-XPRO boards, the second runs just on SAMR21-XPRO boards. Native target hasn't the button.
### Server and Client (Periodic PUT) - native target
Create taps interfaces (to which RIOT will connect). Go to `/dist/tools/tapsetup` and type
```
$ sudo ./tapsetup -c
```
After this step we have created 3 tap interfaces: tap0, tap1 and tapbr0. 
Now, we compile the server. Go to `/examples/iotivity-examples/server` and type
```
$ make all BOARD=native 
```
Run the server by invoking
```
$ sudo bin/native/ocf_server.elf tap0
```
The server output will be
```
RIOT native interrupts/signals initialized.
LED_RED_OFF
LED_GREEN_ON
RIOT native board initialized.
RIOT native hardware initialization complete.

main(): This is RIOT! (Version: 2017.01-devel-13-g2b77e-mattia-Latitude-E6410-pkg/iotivity)
server_oic: Waiting for address autoconfiguration...ipadapter: waiting for server requests...
ipadapter: waiting for multicast requests...
server_oic: LED0 is OFF
LED_RED_OFF
oc_main: Stack successfully initialized
server_oic: Configured network interfaces:
Iface  5   HWaddr: e6:e8:ff:6b:0c:f2 
           
           MTU:1500  HL:64  RTR  RTR_ADV  
           Source address length: 6
           Link type: wired
           inet6 addr: ff02::1/128  scope: local [multicast]
           inet6 addr: fe80::e4e8:ffff:fe6b:cf2/64  scope: local
           inet6 addr: ff02::1:ff6b:cf2/128  scope: local [multicast]
           inet6 addr: ff02::2/128  scope: local [multicast]
           inet6 addr: ff02::1a/128  scope: local [multicast]
           inet6 addr: ff03::158/128  scope: global [multicast]
```
It is listening for requests.
Open a new terminal window and go to `/examples/iotivity-examples/client` and type
```
$ make all BOARD=native
```
Run the client by invoking
```
$ sudo bin/native/ocf_client.elf tap1
```
The client is running and it starts with the discovery phase
```
RIOT native interrupts/signals initialized.
LED_RED_OFF
LED_GREEN_ON
RIOT native board initialized.
RIOT native hardware initialization complete.

main(): This is RIOT! (Version: 2017.01-devel-13-g2b77e-mattia-Latitude-E6410-pkg/iotivity)
LED_RED_OFF
client_oic: Waiting for address autoconfiguration...ipadapter: waiting for server requests...
ipadapter: waiting for multicast requests...
oc_main: Stack successfully initialized
client_oic: Configured network interfaces:Iface  5   HWaddr: a6:ab:89:bd:1f:80 
           
           MTU:1500  HL:64  RTR  RTR_ADV  
           Source address length: 6
           Link type: wired
           inet6 addr: ff02::1/128  scope: local [multicast]
           inet6 addr: fe80::a4ab:89ff:febd:1f80/64  scope: local
           inet6 addr: ff02::1:ffbd:1f80/128  scope: local [multicast]
           inet6 addr: ff02::2/128  scope: local [multicast]
           inet6 addr: ff02::1a/128  scope: local [multicast]
           inet6 addr: ff03::158/128  scope: global [multicast]

client_oic: continue discovery
Outgoing message to [ff03:0000:0000:0000:0000:0000:0000:0158]:5683
client_oic: continue discovery
...
ipadapter: got server request
Incoming message from [fe80:0000:0000:0000:241c:c8ff:fe14:3d79]:56789
ipadapter: waiting for server requests...
client_oic: Discovery done
client_oic: Operate...
LED_RED_ON
```
Once the resource is discovered, the client switches on its LED as notification of Discovery Completed.
From this point it sends a PUT request every second.
Client Output: 
```
client_oic: Sent PUT request
Outgoing message to [fe80:0000:0000:0000:241c:c8ff:fe14:3d79]:56789
ipadapter: got server request
Incoming message from [fe80:0000:0000:0000:241c:c8ff:fe14:3d79]:56789
ipadapter: waiting for server requests...
client_oic: PUT_light:
client_oic: PUT response OK
ipadapter: got server request
Incoming message from [fe80:0000:0000:0000:241c:c8ff:fe14:3d79]:56789
ipadapter: waiting for server requests...
client_oic: OBSERVE_light: key state, value 0
```
Server output:
```
Incoming message from [fe80:0000:0000:0000:a4ab:89ff:febd:1f80]:56789
ipadapter: waiting for server requests...
server_oic: PUT request
server_oic: key: state value: 0
server_oic: LED0 is OFF
LED_RED_OFF
Outgoing message to [fe80:0000:0000:0000:a4ab:89ff:febd:1f80]:56789
server_oic: GET request
server_oic: Light state 0
Outgoing message to [fe80:0000:0000:0000:a4ab:89ff:febd:1f80]:56789
```
TAPs interfaces can be easily deleted. Go to `/dist/tools/tapsetup` and type
```
$ sudo ./tapsetup -d
```

### Server and Client (Periodic PUT) - SAMR21-XPRO target
Now, we reproduce the previous scenario using 2 [SAMR21-XPRO][4] nodes.
Connect your nodes, go to `/examples/iotivity-examples/server` and check the availability by typing:
```
$ make list-ttys
```
The output will be similar to
```
/sys/bus/usb/devices/2-1.3: Atmel Corp. EDBG CMSIS-DAP serial: 'ATML2127031800009004', tty(s): ttyACM0
/sys/bus/usb/devices/2-1.4: Atmel Corp. EDBG CMSIS-DAP serial: 'ATML2127031800008718', tty(s): ttyACM1
```
We will use Serial Numbers in order to identify the designed node during the compilation phase.
Now, we compile the server
```
$ make flash BOARD=samr21-xpro SERIAL=server_node_serial
```
then we open the serial connection
```
$ make term BOARD=samr21-xpro SERIAL=server_node_serial
```
The server starts and it is waiting incoming requests.
Now, we open a new terminal window and go to `/examples/iotivity-examples/client` and type
```
$ make flash BOARD=samr21-xpro SERIAL=client_node_serial
$ make term BOARD=samr21-xpro SERIAL=client_node_serial
```
Client starts the discovery phase. Once it find a resource, it switches on its LED  and it starts with periodic PUT requests. The server LED will blinkClient starts the discovery phase. Once it find a resource, it switches on its LED and it starts with periodic PUT requests. The server LED blinks periodically.  periodically.
Client and Server terminal outputs are the same in case of native target.

### Server and Client_Switch  - SAMR21-XPRO target

[1]: https://github.com/iotivity/iotivity-constrained/
[2]: http://cbor.io/
[3]: https://openconnectivity.org/resources/specifications
[4]: http://www.atmel.com/tools/ATSAMR21-XPRO.aspx
