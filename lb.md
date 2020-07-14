# Configuring Undertow as a dynamic load *balancer*

### Add the mod_cluster Filter to Undertow
The default undertow subsystem configuration does NOT include a mod_cluster filter, even in
the ha and full-ha clustered EAP profiles. This filter has to be created and configured to use
the correct multicast parameters by referring to a socket-binding.

`/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:add(management-socket-binding=http, advertise-socket-binding=modcluster)`


#The two attributes required by a mod_cluster filter are:
- **management-socket-binding:** informs Undertow where to receive connection information
and load balance metrics from back-end web servers. It should point to the socket-binding
where EAP receives HTTP requests, which is http by default.
- **advertise-socket-binding** informs Undertow where to send advertisement messages,
that is, the multicast address and UDP port, by referring to a socket-binding name.


After **creating** the filter, it should be **enabled** in the desired Undertow (virtual) hosts:

`/profile=ha/subsystem=undertow/server=default-server/host=default-host/filter-ref=lb:add`

*Notice lb is the name assigned to the mod_cluster filter defined in the previous command.*

It is also recommended to configure an advertise key shared by the mod_cluster client and
server. To configure the advertise key on the application server instances, that is, on the cluster
members:

`/profile=ha/subsystem=modcluster/mod_cluster-config=configuration:write-attribute(name=advertise-security-key,value=secret)`

**To configure the key on the load balancer server instance:**

`/profile=ha/subsystem=undertow/configuration=filter/mod_cluster=lb:write-attribute(name=security-key,value=secret)`



cluster_M0
------------------
server_1
192.168.99.110:8180
http://192.168.99.110:10090

server_2
192.168.99.110:8280
http://192.168.99.110:10190/
--------------------

#Load Balancer

To configure a new load balancer, the Undertow subsystem must support a reverse
proxy that will load balance the request between two servers. Create a reverse proxy
named cluster-handler:

`[standalone@127.0.0.1:9090 /] /subsystem=undertow/configuration=handler/reverse-proxy=cluster-handler:add `

**Create a new socket binding that will be used by the AJP protocol running on the server named server (172.25.250.254:8109). Name it remote-server1:**

`[standalone@127.0.0.1:9090 /] /socket-binding-group=standard-sockets\remote-destination-outbound-socket-binding=remote-server1:add(host=172.25.250.254, port=8109)`

**Create in Undertow a reference to the socket binding created previously to the server1 and bind it with the scheme AJP and it should access the cluster context path:**

`[standalone@127.0.0.1:9090 /] /subsystem=undertow/configuration=handler/reverse-proxy=cluster-handler/host=server1:add(outbound-socket-binding=remote-server1, scheme=ajp,instance-id=server1, path=/cluster)`


##Important!
*To enable the sticky session, the instance-id attribute should have the same value specified by the jboss.node.name System Property.*


**Create a new reverse proxy location named /cluster referring to the cluster-handler handler:**

`[standalone@127.0.0.1:9090 /] /subsystem=undertow/server=default-server/host=default-host/location=\cluster:add(handler=cluster-handler)`

