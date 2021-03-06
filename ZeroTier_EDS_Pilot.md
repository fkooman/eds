# ZeroTier

ZeroTier is virtual networking technology for creating an "overlay" network 
over the Internet. It is a kind of VPN technology, but optimized to work in
a "peer-to-peer" fashion. Not all traffic between the participants will go 
through a central point.

Depending on the use case it may or may not make sense to deploy ZeroTier. If 
your use case is providing access to a network residing in one place it does 
not offer benefits over a "traditional" VPN. If, however you want to create a 
secure and private network distributed over various locations with a dynamic 
ever changing amount of participants ZeroTier can be deployed.

# ZeroTier EDS Pilot

Exactly for this last use case it was investigated how to deploy ZeroTier and 
integrate it with SURFconext and SURFteams to make it easy to manage the 
participants to the network and allow users to create their own networks.

This is more difficult with a "traditional" VPN as that would typically 
require installing a VPN service and also configuring it. Deploying a ZeroTier 
service is much simpler. 

An extra use case that was investigated is running a number of virtual systems
behind one ZeroTier instance so that those virtual systems could benefit from
access to the network without requiring the installation of additional 
software.

## SURFconext

With SURFconext the identity federation as ran by SURFnet is meant. It is a 
service to use one's organization credentials to authenticate to various 
services around the Internet without providing your credentials directly to the
service using the SAML technology.

## SURFteams

SURFteams is the service, part of SURFconext, that deals with groups and 
their members. A user can create a group and manage its members. This 
information is exposed to the service. The service can use this information 
for authorization purposes, e.g.: is the current logged in user allowed to 
access this virtual network.

# Architecture

ZeroTier is not only software, the software is called ZeroTier One, but also 
a management service called ZeroTier Central. The software is available as 
free software for Windows, Linux and macOS. The mobile applications are 
proprietary.

A user can create a network in ZeroTier Central and add users to it. The 
authentication to the ZeroTier Central portal uses "traditional" user name and
password authentication. There is currently no support for group membership to
allow automatic memberships of networks based on groups.

The ZeroTier One software can operate in three different modes. It can be a 
"node", which means an end device for connecting to a "network controller", 
operating either in "leaf" mode, or in "bridge" mode for relaying traffic to 
networks behind the ZeroTier node. The third mode is "root" that is used 
for discovery and signaling for network controllers and nodes, mostly to deal
with difficult networks deploying e.g. NAT or firewalling. 

For the pilot it was decided to run a network controller, allowing for 
creating and managing networks and using SURFconext and SURFteams for 
user access. The official ZeroTier root nodes will be used for this pilot as 
that will make it possible to use the free software as well as the mobile 
applications without modifying them. 

# Design

The ZeroTier network controller has an API that can be controlled using HTTP 
calls for managing its networks. Every network has a configuration that 
among other information includes an IP range that will be assigned to clients
a list of members.

All participants in the ZeroTier network have a 10 digit unique 
identifier. Claiming this identifier requires access to a private key that 
is generated by each participant. So identifying a node or controller with its
10 digit identifier is sufficiently secure. 

A network controller identifies networks by its 10 digit identifier and an 
additional 6 digit network identifier. In order to allow access to a network, 
the client identifier needs to be registered in the network controller and the
client needs to join the network using it 16 digit identifier.

Managing this can all be wrapped behind a web interface to make this easier. 
This is exactly what was done for the pilot.

Due to the limitations of the technology, regarding integrating with SURFconext
and SURFteams, the flow for the user is a bit complicated and require some 
additional steps compared to ZeroTier Central.

To create a network, the user has to do the following:

1. Go the pilot portal;
2. Authenticate using the credentials from the institute (using SURFconext);
3. Register the client identifier(s) in the pilot portal by copy/pasting the 
   client identifier from the client running on the user's device;
4. Create a network and choose a group (as created in SURFteams) to bind this 
   network to;
5. Use the client to (manually) join the network just created;

Users that want to be part of this network would have to go through the same
steps, but they skip step 4.

If no group exists, this group needs to be first created in SURFteams and its 
members invited to join the group. 

# Demo 

A demo setup is available!

1. Go to [https://zerotier.tuxed.net/](https://zerotier.tuxed.net/);
2. Choose your identity provider, or _onegini_ if you do not have 
   access to one to create a new account there;
3. Register your ZeroTier clients in the 
   [Account](https://zerotier.tuxed.net/portal/account) tab, with the 
   identifier(s) you can find in the GUI, or by using 
   `zerotier-cli info` from the CLI;
4. Click the "Obtain group membership" on the Account tab, or click 
   [here](https://zerotier.tuxed.net/portal/_voot/authorize) (**NOTE**: this 
   will be automated later so that this step is no longer needed);
5. On the [ZeroTier](https://zerotier.tuxed.net/portal/zerotier) tab you can
   find networks you created, or networks you have access to;

On the [ZeroTier](https://zerotier.tuxed.net/portal/zerotier) tab you can 
create a network at the bottom of the page, choose a group to "bind" the 
network to. If you cannot choose a group, create one 
[here](https://teams.connect.surfconext.nl/).

The client IDs you registered will by automatically _authorized_ to join the 
networks you have access to, but you will still need to manually join the 
network in your ZeroTier client. To join a network you can use the listed 
_Network ID_ to do that by providing that to your ZeroTier client in the GUI, 
or the CLI using `zerotier-cli join <Network ID>`.

# Code

The project contains various parts:

- [ZeroTier](https://zerotier.com/), with network controller enabled;
- [Portal](https://github.com/eduvpn/vpn-user-portal/tree/zerotier);
- [Backend](https://github.com/eduvpn/vpn-server-api/tree/zerotier);

RPM packages are created for Fedora and CentOS and available 
[here](https://copr.fedorainfracloud.org/coprs/fkooman/zerotier/), the 
review request for inclusion in Fedora and EPEL is 
[here](https://bugzilla.redhat.com/show_bug.cgi?id=1352169).

The packages have the network controller enabled by default.

# TODO

These are items still to be done for the pilot:

1. Obtaining group memberships automatically on login is not done yet, this 
   needs to be automated, currently this must be manually triggered. By going 
   to [https://zerotier.tuxed.net/portal/_voot/authorize](https://zerotier.tuxed.net/portal/_voot/authorize);
2. Allow for choosing a user identifiable name when adding a client on the 
   account page to be able to recognize them;
3. Allow setting that a particular client is a bridge;
4. Allow removing clients and networks;
