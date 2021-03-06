---
layout: page
title: User Guide
---

{% include toc.html %}

This guide describes how users log into XOS, acquire VMs, and access
XOS services. It assumes the reader is familiar with the general
concepts presented in the [Architecture Guide](/archguide/). 

Because this guide is focused on users, we use OpenCloud as an example
of an operational cloud that users might access. This implies a
particular portal, a particular set of nodes and sites, and a
particular collection of services. Substitute local specifics for an
alternative XOS installation.

## Getting Started

Users access OpenCloud by logging into the portal at
[portal.opencloud.us](http://portal.opencloud.us). New users register
through the same portal, with the request approved by the PI
(Principle Investigator) at their home site. Sites join OpenCloud
through an off-line process that starts by sending email to
<support@opencloud.us>.

Once the user has an account, the following steps are a quick guide
to utilizing OpenCloud resources:

1. **Upload a public key.** Select the *Users* tab in the left-hand
   navigation bar, click on your email address, and then paste your
   public key in the box. Click the *Save* button at right.

2. **Ask your Site PI to create a slice.** Instructions for PIs (see
   [Administering a Site](#admin-site) for more information):
   - Select the *Slices* tab in the left-hand navigation bar, and then
     the *Add Slice* button on the right. In the *Slice Details* form,
     choose a name for the slice and select your own site from the
     drop-down menu. Slice names must be unique. Click the *Save and
     continue editing* button on the right before proceeding.  
   - Select the *Privileges* tab from the top, and then click the *Add
     another slice privilege* link.  Add the users that will
     administer the slice with the Admin privilege. (These users will
     be able to extend slice privileges to other users.)  
   - Click the *Save* button when done.

3. **Create instances.** Use either the [Tenant](#tenant-view) or the
   [Developer](#developer-view) View (see next section) to instantiate
   instances (VMs) for your slice.

4. **Log into instances.** Once an instance comes up, you will be able to
   learn its *instance ID*. You can then ssh into the instance using
   this ID and the physical node it is running on. See
   [Accessing a Instance](#access-instance) for more information.

## User Views

Users interact with OpenCloud through a configurable set of *Views*,
each tailored for a different usage scenario or workflow. By default,
a user's home dashboard includes the *Tenant* view; a *Customize* tab
allows the user to add and remove views from their home dashboard,
including the more "advanced" *Developer* view. Both the Tenant and
Developer views are used to administer a user's slices. From any page,
selecting the *Home* tab in the left-hand navigation bar brings the
user back to his or her home dashboard.

Users are also able to directly navigate the underlying data model
using the left-hand navigation bar. The top set of tabs (Deployments,
Sites, Slices, and Users) correspond to the core XOS objects, as
described in the [Data Model](/archguide/#data-model). The
following sections of this guide describe how administrators use these
tabs to manage users, sites, and deployments, respectively. (The views
described in this section are typically use to manage slices.) The
bottom set of tabs (RequestRouter, HyperCache, and Syndicate)
correspond to services that extend XOS. Most users will have no need
to directly access these services, but will instead indirectly access
the services through the tailored workflows supported by the various
views.

There are also views designed to support operators, as described
in the [Operator Guide](/opsguide/).

### Tenant View

The Tenant view provides a simple means to acquire Instances, with
minimal control over the low-level details of where those Instances are
placed and what networks interconnect them. It is loosely patterned
after the Amazon EC2 interface.

The Tenant view is limited to the ViCCI Deployment, which includes
clusters at five sites throughout the US and Europe. Users that want
to acquire Instances on any other deployment must use the Developer
view.

The Tenant view allows users to specify the number of instances that are
to be instantiated on the available sites, select an *Image* and
*Flavor* for each instance, mount select *Data Set(s)* into each instance,
and specify the *TCP Ports* the slice is going to use. Users can also
set the *ServiceClass* for the slice, although *Best Effort* is
currently the only supported class.

In the Tenant View, click on the *SSH Commands* button to see SSH
commands that can be cut-and-pasted into the terminal for logging into
your instances; click the *Download* button to save them as a local
text file.  [Accessing an Instance](#access-instance) explains other 
ways to configure SSH access for a slice's instances.

Note that slices intially created through the Tenant view may also be
managed through the Developer view. The reverse is true as long as the
slice is instantiated only on the ViCCI deployment.

### Developer View

The Developer view gives users full control over how their slices are
instantiated, including instance placement, network configuration, and
the privileges granted to other users:

* **Instances:** Select the *Instances* tab at the top to manage the
  instances bound to the slice. From the resulting page, select a target
  *Deployment*, and then pick an individual *Node* at that deployment. 
  Users are also able to select an *Image* and a *Flavor* on either a
  per-instance or a per-slice basis. All deployments that the user is
  permitted to access are visible when instantiating instances.

* **Networks:** Select the *Networks* tab at the top to manage the
  networks connected to the slice. Each slice is automatically
  configured with two virtual networks (one public and one private),
  but the user has the option of connecting the slice's instances to
  additional virtual networks. (Creating additional networks and
  connecting a slice to them is currently an undocumented feature.)

* **Privileges:** Select the *Privileges* tab at the top to manage the
  users that have access to the slice. Granting a user *Admin*
  privilege means giving them the authority to modify slice
  parameters. Granting a user *Default* privilege means the giving
  them access to the slice itself, that is, the right to ssh into the
  slice's instances.

### xsh View

The xsh view provides an interactive shell through which users can
access XOS objects. It is a Javascript-based environment that includes
*xoslib*, a library projection of the XOS data model. A builtin
tutorial illustrates how to use xsh.

## Accessing an Instance

Instances connect to the network via NAT; logging into the instance relies
on SSH proxying to forward incoming SSH connections. In the Tenant View, 
click on the *SSH Commands* button to see SSH
commands that can be cut-and-pasted into the terminal for logging into
your instances. In the Developer
View, the instance Id and node name are displayed in the Instance frame.
You can use this information to add lines to your ~/.ssh/config file 
similar to the following:

```
Host foobar
  User ubuntu
  IdentityFile ~/.ssh/id_rsa
  ProxyCommand ssh -q instance-0000006c@node5.cs.arizona.edu
```

In the above, replace "foobar" with a label of your choice for this
instance.  *User* is the default login user for the image.
*IdentitiyFile* should point to the key that you've uploaded to
OpenCloud.  *ProxyCommand* should point to the instance ID and node
for the instance. Once an entry is present for the instance in
~/.ssh/config, you can login using the label:

```
# ssh foobar
```

Other utilities like scp also work as expected when referencing
the instance using the label.

A current limitation is that only one user key is injected into the
slice. Because SSH is indirect through the *ProxyCommand*, it is not
sufficient to manually add additional keys for other users to an 
account inside the instance; for the time being, an administrator will 
need to add the additional keys to the proxy environment as well.

In addition to SSH connectivity via NAT, there are two other
network-related issues of note. First, to run an Internet-accessible
service in a slice, it is necessary to reserve a TCP or UDP port.
This is done using the *Network Ports* field in the Tenant View. The
service can then be accessed at this port on the hosting server (e.g.,
*node5.cs.arizona.edu* in the above example). Second, all the instances
at a given site are automatically connected by a private network. Run
*ifconfig* from within an instance to learn the instance's private address
(i.e., the *10.x.x.x* address associated with *eth0*). This private
virtual network is per-site. Instances in different sites must use an
Internet-accessible address to communicate (i.e., using a reserved
port and hosting server name as described above).

## Administering a User

All users are able to manage their own accounts and Site Admins are
able to manage the accounts of users homed at that site. Select the
*Users* tab in the left-hand navigation bar, and then click on the 
desired user. The available user details are as follows:

* **Login Details:** Select the *Login Details* tab to set parameters
  of the user's account. These include the user's *Email Address*
  (which uniquely identifies the user), home *Site*, *Password*, and
  *Public Key* (which is used to ssh into any instances created on the
  user's behalf). Click the The *Is Active* button to enable the
  account.

* **Contact Information:** Select the *Contact Information* tab to set
    the user's name and phone number.

* **Site Privileges:**: Select the *Site Privileges* tab to set the
  site-related privileges for the user.

* **Slice Privileges:**: Select the *Slice Privileges* tab to set the
  slice-related privileges for the user.

## Administering a Site

Site Admins are responsible for managing the users, slices, nodes, and
deployments affiliated with the site. Select the *Sites* tab in the
left-hand navigation bar and click on the desired site. The available
site details are as follows:

* **Users:** Select the *User* tab to proactively add users to a site,
    as well as change information for existing users.  Alternatively,
    a Site Admin is informed via email when a user requests an
    account, and has the opportunity to either accept or deny the
    request.

* **Privileges:** Select the *Privileges* tab to grant users at the
  site Admin, PI, or Tech privileges. All users homed at the site have
  default privilege.

* **Slices:** Select the *Slices* tab to create and manage the site's
  slices. To create a new slice, select *Add Slice*, fill in the
  *Slice Name*, and select *Save and Continue Editing*. From the
  resulting page, fill in the relevant slice details, and select the
  *Privileges* tab to define the set of users that are to have access
  to the slice. Alternatively, a site Admin can administer the site's
  slices by selecting the *Slices* tab in the left-hand navigation
  bar, as described in [Getting Started](#getting-started).

* **Nodes:** Select the *Nodes* tab to create and manage the site's
  nodes. To create a new node, select *Add Node*, fill in its FQDN
  and the *Deployment* it belongs to.

* **Deployments:** Select the *Deploymements* tab to affiliate the
  site's nodes with one or more deployments.

## Administering a Deployment

Deployment Admins are responsible for managing the privileges, sites,
images, flavors, and visibility for the deployment. Select the
*Deployments* tab in the left-hand navigation bar and click on the
desired deployment. The available deployment details are as follows:

* **Images:** The *Images* selector is used to specify which images
  are supported by the deployment. Root adminstrators specify the
  global set of available images.

* **Flavors:** The *Flavors* selector is used to specify which flavors
  are supported by the deployment. Root adminstrators specify the
  global set of available flavors.

* **AccessControl:** The *AccessControl* form is used to specify a
  policy for what users are and are not allowed to access the
  deployment's resources. A given user sees only those deployments
  that have granted access when they attempt to instantiate
  intances. The current policy language is simple: *allow all*
  indicates that all users may instantiate instances on the deployment,
  *allow site &lt;sitename&gt;* grants access to all users from a
  particular site, and *allow user &lt;email&gt;* grants access to a
  specic user.  Deny rules may also be used (*deny site
  &lt;sitename&gt;*, *deny user &lt;email&gt;*, etc).  Rules are
  evaluated in order from top to bottom, with an implicit *deny all*
  at the bottom of the list.

The *Privileges* tab is used to grant other users Admin privileges for
the deployment.

The *Sites* tab is used to bind a Site (and the Nodes they host) to
this Deployment. Nodes are imported into a Deployment through a
*Controller* that is responsible for instantiating and managing the
Nodes. For example, in the case of an OpenStack cluster, the
Controller effectively connects XOS to the Nova, Neutron, and Keystone
services running on the OpenStack head node.

To create a new Deployment-to-Site-to-Controller binding, use the *Add
another Site Deployment* link at the bottom of the Sites tab. You will
be prompted for the Site and the Controller. If an existing Controller
is not suitable, then the "+" button next to the Controller dropdown
may be used to create a new Controller.

Creating a Controller involves defining the following fields:

* **Name:** A human-readable name for the deployment. 

* **Backend Type:** The type of backend for the deployment. Current
    backend types are limited to "OpenStack".

* **Version:** The version of the backend. This is currently limited
    to *Icehouse* and *Havana*.

* **Auth URL:** Controller-specific authentication URL. For OpenStack
    controllers, this is the URL of the Keystone endpoint.

* **Admin User:** Controller-specific admin username. 

* **Admin Password:** Controller-specific admin password.

* **Admin Tenant:** Controller-specific admin tenant. 

The list of controllers is not a top-level item in the navigation
panel, so to edit an existing Controller, first go to another object
(e.g., Deployment), and then use the *Core* link at the top of the
page to bring up the list of Core XOS objects. Select the *Change*
link next to *Controller*. *[Workflow to be cleaned up.]*

## Administering Images

To upload an image to XOS, place the image file in /opt/xos/images. Note that
adding a file to this directory must be done atomically, for example by uploading
the file elsewhere and then using 'mv' to move the file into the directory. If
a file is uploaded directly to /opt/xos/images, then it's possible the Synchronizer
may attempt to sync the image to controllers while the upload is in progress,
before the file is complete.

Once the image has been placed in /opt/xos/images, the Synchronizer will run and
automatically sync that image to all controllers. 

Even though an image has been uploaded and synced, it is still not available
for use until it has been enabled in a deployment. See the Administering a
deployment section above. 

## Services

XOS is designed to support a set of contributed services, but the set
of services available on any particular cloud is as cloud-specific as
the underlying hardware infrastructure. In the case of OpenCloud,
there are currently three contributed Services, each of which extends
the XOS Data Model with its own abstract objects, and hence, is
accessible via OpenCloud's REST API. This section briefly describes
these three services (for illustrative purposes), and concludes by
describing how to prototype a new service.

### Syndicate

Syndicate is a scalable storage service. It provides a private shared
volume that both instances and users can locally mount as a read/write
filesystem. Syndicate also offers public read-only volumes that
contain popular scientific datasets. Slices can specify that they want
to mount one of these datasets (public volumes).

Support for enabling a private volume and selecting public datasets is
built into the Tenant view, meaning users need not directly access
Syndicate through the corresponding tab in the left-hand navigation
bar. When accessed in this way, the user will see the slice's private
volume mounted under */syndicate* when they log into each of their
instances.

Selecting the *Syndicate* tab in the left-hand navigation bar and then
clicking on the *Volumes* menu option allows users to control specific
volume parameters. This is a necessary step when using the *Developer*
view. From the resulting page, users can add and remove additional
volumes to their slices, bind their volumes to other slices, and
control volume capabilities for other users and slices.

#### Mounting to an Existing Volume in a Slice

A user can add another volume to a slice in the following manner (the
user must start from the Volumes menu).

1. Click the desired volume.

2. Select the *Slices* tab.

3. Click *Add another Volume Slice*.

4. Fill in the volume parameters, as follows:
    - **Slice**: The slice to receive the volume.
    - **Cap read data**: Check this box if an instance should be
      permitted to read volume data.  If unsure, check this box.
    - **Cap write data**: Check this box if a instance should be
      permitted to write volume data.  If unsure, do not check
      this box.
    - **Cap host data**: Check this box if an instance should be 
      permitted to host data for the volume. Only check this 
      box if you can rely on the instances to work with Syndicate
      to keep the volume data consistent and available.  If 
      unsure, do not check this box.
    - **UG port**: This is a port number, between 1024 and 65534,
      on which each instance's Syndicate User Gateway will listen.
      It does not matter what number the user chooses; it only 
      matters that it is available on each instance in the slice.
    - **RG port**: This is a port number, between 1024 and 65534,
      on which each instance's Syndicate Replica Gateway will listen.
      It does not matter which number the user chooses; it only 
      matters that it is available on each instance in the slice.
5. Click *Save*.

Once these steps are carried out, Syndicate ensures that the new
volume is automatically mounted under */syndicate/* in each instance.

#### Creating and Updating Volumes

Users can create or change the parameters of their volumes by
selecting their volume from the volume list, and navigating to the
*Volume Details* tab.  Most of the *Volume Details* fields are
self-explanatory, but a few warrant additional comment:

* **Blocksize**:  This is the number of bytes in a block. 
In Syndicate, a block is the smallest unit of transfer, storage, 
and data consistency.  Smaller values reduce the number of 
bytes to invalidate on a write, but larger values improve 
read performance by reducing the number of data transfers.
A good value for read/write volumes is 102400 (100KB).  Read-only
volumes should choose the medium file size as the block size,
to maximize transfer efficiency.

* **Archive**:  If set, this volume will need to be configured 
such that a Syndicate Acquisition Gateway (AG) can populate it 
with dataset metadata.  Do not check this box unless you know 
what you are doing.

* **Cap host data**:  If set, this means that by default, an instance can 
help Syndicate replicate data and coordinate writes to it.  This 
implies a larger degree of trust in the instance.  If your instances 
will do most of the I/O in your volume, you should check this box.
If you or other machines outside of OpenCloud will do most of the I/O,
you should *not* check this box.

#### Volume Access Control

Users can authorize other users to access their volumes. This is not
limited to within OpenCloud. If Alice permits Bob to access her
volume, then Bob can do so from his OpenCloud instances or from his
personal machines. (Likewise, Alice can share her OpenCloud volume
with her own off-OpenCloud machines.)  OpenCloud access to a Users can
control the capabilities other users will have: read access (*Cap read
data*), write access (*Cap write data*), and host access (*Cap host
data*).

Read and write access is self-explanatory. Host-access should only be
granted if the user is sufficiently trustworthy, since the *host data*
capability will grant that user the ability to coordinate writes to
the volume's files.  If unsure, the user should not check *Cap host
data*.

### HyperCache

HyperCache (HPC) is a Content Distribution Nework (CDN). It uses a set
of distributed caches to to accelerate the delivery of content on
behalf of a content provider. Selecting the *HyperCache* tab in the
left-hand navigation bar reveals a menu of options that govern how
content is to be delivered via the CDN. From the resulting pages,
users can add/remove origin servers that source content, register URLs
that name content, and specify where content is (and is not) to be
served.

* **Service Provider:** A CDN administrator. Each service provider
  manages (creates accounts for) a set of content providers.

* **Content Provider:** The primary "tenant" of the CDN. All other
  parameters are controlled by a given content provider. Each content
  provider specifies a set of *Users* that are allowed to act on its
  behalf, and registers one or more *CDN Prefixes* that will name its
  content (see below).

* **CDN Prefix:** The FQDN prefix of URLs that names content to be
  delivered via the CDN. Each CDN Prefix has a default *Origin Server*
  that HPC uses to acquire content (see below).

* **Origin Server:** The URL of an origin server that sources content.
  Each origin server is bound to one CDN Prefix, but one CDN Prefix
  can name content sourced at multiple origin servers. Typically, a
  particular origin server is directly or indirectly identified in the
  URL, but there is an explicitly specified default origin server that
  HPC uses when this is not the case. The content provider also
  specifies the *Protocol* used to download content from an origin
  server (typically, this is set to HTTP), and whether or not requests
  that cannot be handled by the CDN should be redirected to the origin
  server (typically, this setting is enabled).

* **Site Map:** A file that specifies a policy for how requests are to
    be redirected.

* **Access Map:** A file that specifies a policy for where content is
    and is not delivered.

The format of the map files is defined elsewhere.

### RequestRouter

RequestRouter (RR) is a multi-tenant service that redirects user
requests to the best instance of a given client service. For example,
HyperCache uses RR to redirect HTTP GET requests to the best cache to
serve content to a particular end-user. RR determines "best" according
to a client-specified policy known as a service map.

Selecting the *RequestRouter* tab in the left-hand navigation bar,
clicking the *Service Map* menu item, and then selecting a particular
map allows users to specify the policy that guides request redirection.
From the resulting page, users can register a client service, register
a URL that name the service, and configure various maps.

* **Name:** Name of the service map.

* **Owner:** Client service that owns the service map.

* **Slice:** Slice in which the client service runs. Indirectly
  identifies the candidate service instances to which RR redirects
  requests.

* **Prefix:** The FQDN prefix of a URI that is to be managed by
    RequestRouter. This prefix effectively names the service in a
    server-independent way.

* **Site Map:** A file that specifies a policy for how requests are to
    be redirected.

* **Access Map:** A file that specifies a policy for where the service
    is and is not available.

The format of the two map files is defined elsewhere.

### Prototyping a New Service

Users are able to create their own services, and make them available
to other users. There are two approaches to doing this, which we refer
to as *prototype* and *production*. A prototype service, as its name
suggests, is usually in development phase. XOS provides a set of
light-weight mechanisms to help the developer manage a prototype
service. Similarly, a production service is one that is officially
offered as part of a deployment, for example, OpenCloud offers three
production services: RequestRouter, HyperCache, and Syndicate. Section
[Adding Services](/devguide/addservice/) of the
Developer Guide describes how to add a production service to XOS. This
section focuses on the available mechanisms for prototyping a service.

## Mailing Lists

All registered OpenCloud users are automatically subscribed to
[users@opencloud.us](mailto:users@opencloud.us). Use this mailing list
to announce topics of general interest to other users, and to ask
questions about best practices using OpenCloud.

To report problems or ask for specific assistance, send email to
[support@opencloud.us](mailto:support@opencloud.us).

