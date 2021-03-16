.. _KarbonObjectBootcamp:

.. role::   raw-html(raw)
      :format: html

Advanced Bootcamp (Automation)
==============================

Topics
------
-  Nutanix Karbon

-  Nutanix Object

-  Nutanix Era

Goal
----

Setup a fully working NextCloud solution, highly available, hosted on a
Kubernetes cluster.

The MariaDB database backend will be deployed and protected using
Nutanix Era solution.

An object storage solution, deployed with Nutanix Object will be used as
an external repository, setup on the NextCloud platform.

Design
------

Global architecture

|image0|

Kubernetes Storage Access

|image1|

Deployment
----------

Connect to your Frame Desktop, using the Frame Jumphost URL, your Frame
User and the Frame Jumphost Password as provided in Ressources lab
document. You’ll stay on this Jumphost for all the lab.

Please change the keyboard layout to UNI, this will use your keyboard
layout.

|image2|

In the Lab guide we use the word “burger” It’s the menu start on the top
left of Prism Central, which look like this |image3|

Please use the **Google Chrome** browser in the Frame Jumphost for the
entire lab.

As the keyboard mapping, or copy / paste something has some strange
behavior, some scripts / text are available on an AWS Bucket. It will be
indicated how to get it on the documentation when needed.

Objects
-------

**This is a showcase, do not click on create, and click cancel afterwards, as an object store has 
already been created for you.**

-  With Google Chrome, **connect** to the **Prism Central** (as provided
   in Ressources lab document), click on the\ |image4| Burger Menu,click 
   **Services**, click **Objects**
-  Click **Create Object Store** / Continue
-  Enter an object store name based as provided in Ressources lab
   document / next
-  Select performance (Estimated) to Custom. It will deploy a very small
   object instance to save cluster resources for other LAB
   participants.(don’t change vCPU or Memory)
-  Set 100 GiB as capacity / Next
-  Select **your corresponding** Nutanix Cluster
-  Select Managed-Network for both Object Infra Network and Objects
   Public Network. Enter the Objects Infra IPs and Object Public IPs
   with the information provided in Ressources lab document. 

.. hint::

    The creation of an object store is time-consuming and not feasible 
    in this time with parallel staging

Karbon/Kubernetes Cluster
-------------------------

-  Go to **Prism Central**, click on the |image5|\ Burger Menu, click
   Services, click Karbon
-  Click **create a Kubernetes Cluster**
-  Select **Production Cluster** / Next 

-  **Node-Configuration**
  
   -  Enter a name, as provided in Ressources lab document, and select
      **your corresponding** Nutanix Cluster / Next 
      **(leave k8s and Host OS out for now)**

-  **Network**

   -  Select the network named Managed-Network

   -  Enter a master VIP Address (as provided in Ressources lab
      document) / Next

   -  Keep the Flannel Network Provider

   -  Note:(Calico is supported as well, but requires additional
      steps).

   -  Keep the CIDR range by default / Next

-  **Storage-Class**

   -  Select once again **your corresponding** Nutanix Cluster

   -  Enter the cluster username (admin) and password (Nutanix Password
      in the ressources lab document) / Create. Click only once on
      the create button, and wait the popup to be closed, otherwise,
      you’ll deploy multiple time the cluster and the deployment will
      fail!!!

.. hint::

    Wait for the deployment completion. It will take around 15 minutes to 
    complete, go to the next section in the meanwhile

Era
---

-  Open a new tab, and access the Era IP Address Server as provided in
   Ressources lab document

-  Click to Dashboard on the top left section / Databases

-  Click Source / Provision / MariaDB

-  Enter the following information

   -  Create New Server

   -  Database Server Name : mariadb-*yourinitial*-01

   -  Software Profile : Select the only one available

   -  Compute Profile : DEFAULT_OOB_COMPUTE

   -  Network Profile : MariaNW

   -  SSH KEY : Select Text, and copy paste the following string (it’s a
      one line text!)

.. code-block:: Bash
    
    ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAiC8r6cLFLn/c/iR8TKXQhN20wUQwua8DSZM7rpGwuxbgLSSznW/hEVIogx3UoRamU3lIDsD8QKLBiHg29xc/PvR/Ro5Fxvhih3XOQTC14cEwPvgXgMHgPBJ5Vw+bW3a8HVM3S4dsaCsYAkDeHJmXP4G7HN4vrqc3fjb1UYV3iUe8AcheKzD7sG8MSjFBPc7WVI0I47Ly/eKVxVp0csE0fUH6IogUMqA1zp/C/uziAG1vZO6Td2S/FW70OKnCnnNRN8+e7BNlrIuy/0fLsKjUeNEgr8iuFFDoPA23vaPzcZR3hbsICOw7yoFbAsL+z+Mc6O74Nj7bT6WX3rVgMCFFYQ==

https://karbon-bootcamp.s3.eu-west-3.amazonaws.com/ssh-public-key.txt

https://karbon-bootcamp.s3.eu-west-3.amazonaws.com/ssh-public-key.txt

-  Click next, enter the following information

   -  MariadDB Instance Name : nextcloud-*yourinitial*

   -  Database Parameter Profile : DEFAULT_MARIADB_PARAMS

   -  ROOT password : nx2Tech123!

   -  Name of Initial Database : nextcloud

-  Click next, enter the following information

   -  Name : *yourinitial*\ \_nextcloud_TM

   -  SLA : DEFAULT_OOB_GOLD_SLA

-  Click Provision

-  Do not close the browser TAB

.. hint::

   It will take around 10 minutes to deploy the MariaDB Database Please
   proceed to the next section

Kubernetes Setup
----------------

-  Go to Prism Central, click on the Burger \ |image4|, Services, 
   Karbon

-  Cluster Clusters / Check your cluster name / Actions / Download
   Kubeconfig. Click on the Download link

-  Save it to ~/Downloads folder leave file name unchanged)

-  Open the file with Notepad, and copy the content of this file

-  Connect to the linux jumphost (with putty as username: yourusername
   (example userxx) pw: nutanix/4u)

To avoid to write every time the **kubectl**, will create an alias

.. code-block:: Bash
    
    alias k=kubectl


Create a folder to host the kubectl config file

.. code-block:: Bash
    
    mkdir ~/.kube

Use **vi** to configure the kubectl config file on the
linux jumphost

.. code-block:: Bash
        
    vi ~/.kube/config

Open the Downloaded kubeconfig file
(~/Downloads/karbon-<TLA>-01-kubectl.cfg in notepad
copy and paste the text) in the following file

.. hint::

Test the kubetcl configuration, an output should be shown

.. code-block:: Bash
    
    k get pods -A
    
Create a folder named metallb

.. code-block:: Bash
    
    mkdir metallb; cd metallb

Install the metallb service with the following commands

.. code-block:: Bash
    
    kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.5/manifests/namespace.yaml
    kubectl apply -f https://karbon-bootcamp.s3.eu-west-3.amazonaws.com/metallb.yaml
    kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"

Create a file named metallb-config.yaml with the following content,
be careful to adapt the last line with the information as provided
in ressources lab document, for the field **Karbon MetalLB Pool**

.. code-block:: Bash
        
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - x.x.x.x-y.y.y.y
  
https://karbon-bootcamp.s3.eu-west-3.amazonaws.com/metallb-config.txt

-  Configure the metallb setup
  
.. code-block:: Bash
    
    
    k apply -f metallb-config.yaml

Test the current setup, by deploying a basic nginx container

.. code-block:: Bash
    
    k create deployment nginx --image=registry.gitlab.com/fabrice.krebs/nutanix-ch/nginx

Check if the deployed worked

.. code-block:: Bash
    
    k get pods

Expose the deployment behind the metallb load balancer

.. code-block:: Bash
    
    k expose deployment nginx --name nginx --type LoadBalancer --port 80

Get and copy the external IP of the nginx service

.. code-block:: Bash
   
    k get svc

Open a second browser tab and past the IP address. The nginx webpage
should appear. If the test is successful, continue. Otherwise,

.. hint::

If your EXTERNAL-IP is listed as Pending, this indicates an issue with your MetalLB ConfigMap file.
Run kubectl describe configmap config -n metallb-system to verify your IP addresses are correct
Fix your metallb-config.yaml file and run kubectl apply -f metallb-config.yaml again

MariaDB:
--------

Now the MariaDB database server should be deployed. We will need to
retrieve the IP Address from the Era interface. Go back to the Era
Browser Tab:

-  Click on Era text on the Top Left corner

-  Click Dashboard on the Top Left corner / Database / Sources

-  Click on your database server name

-  Under section Database Server VM on the middle of the page, copy the
   IP Address or write it somewhere. We will need it later

NextCloud deployment
--------------------

-  Create a new nextcloud deployment

.. code-block:: Bash
    
    
    k create deployment nextcloud --image=registry.gitlab.com/fabrice.krebs/nutanix-ch/nextcloud

Expose the new deployment to the public network

.. code-block:: Bash
    
    
    k expose deployment nextcloud --type=LoadBalancer --name=nextcloud --port=80 --target-port=80

Retrieve the External-IP address of the deployment

.. code-block:: Bash
    
    
    k get services

Open a new tab and type the external-ip address. You should have the
nextcloud home page available.

-  Do the setup with the following information :

   -  Username : admin

   -  Password : nx2Tech123!

-  **Do not click on Finish yet**

.. hint::

   If you pushed too fast k delete deployment nextcloud ;-)

-  Click on Storage & database / MySQL MariaDB |image6|

   -  Database user : root

   -  Database password : nx2Tech123!

   -  Database name : nextcloud

   -  Replace localhost with the Database IP Address retrieved
      previously

   -  **Unckeck install recommended apps,** as it will take some time
      for applications to be deployed

Click Finish. The initial setup will proceed in a couple of minutes.
You’ll then be able to access the freshly deployed nextcloud.

-  **Do not close the browser TAB**

As the application is still initializing as a background task, the
interface will be a bit slow for a couple of minutes. We will now go to
the next section to create an Object Store bucket, and use it from the
NextCloud application.

Nutanix Object creation
-----------------------

Return to the prism central interface

-  Click on the Burger / Service / Object

-  Generate an access key by clicking on Access Keys on the top / Add
   People / Add people not in a directory service

   -  Email address : your-initial@demo.com

   -  Name : Your name

-  Click Next / Generate Keys / Download Keys (very important as you can
   get it only once)

-  Click on Object Stores on the Top / Click on your cluster

-  Write down somewhere one Object Public IP assigned from the Existing
   Object Store, we will need it for the nextcloud configuration.

-  Click on Create Bucket

   -  Name : nextcloud-yourinitials

   -  Check Enable versioning

   -  Click create

-  Click on the newly created bucket

-  Go to User Access on the left / Edit User Access

-  Search for people your-initial@demo.com

-  Check permission Read and Write / Save

Add Object Storage to NextCloud 
-------------------------------

Go back to NextCloud Tab:

-  Click on the A on the top right section / Apps

-  Go at the bottom of the windows to find External storage support,
   click on Enable

-  Click on the A on the top right section / Settings

-  On the left side, click on External Storages under the
   **Administration Section** (and not the first Personal section)

   -  Folder Name : external_storage

   -  External Storage : Amazon S3

   -  Authentication : Access Key

   -  Bucket : nextcloud-yourinitals

   -  Hostname : The Object Public IPs you’ve copied previously

   -  Keep Enable SSL unchecked

   -  Keep Enable path Style unchecked

   -  Check Legacy (v2) authentication

   -  Select admin user

   -  Access Key : The access key located on the file you’ve downloaded
      when configuring object

   -  Secret Key : The secret key located on the file you’ve downloaded
      when configuring object

   -  Click on the |image7| icone to verify and validate

-  Now the Object storage is connected, let’s try to upload some files.
   Click on the folder icone on the top left section

-  Click on external storage folder

-  Click on the |image8|\ icone on the top section, and upload a couple
   of files from the local computer. Wait for the upload to be done.
   You should see the uploaded file, which aren’t located on the
   Nextcloud itself, but store on the external object store

Check the Embedded Nutanix Object Browser
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  With a Web Browser, connect to the Object Public IP Address of the
   Object Store, used when creating your bucket
   (http://x.x.x.x/objectsbrowser/)

-  Enter the Access Key and the Secret Key you’d previously downloaded

-  Check if your uploaded files appear in the bucket to verify your
   configuration/setup

Additional Lab
--------------

If you have time, a couple of additional steps can be done to have a
good overview of the Nutanix solution.

Check Karbon scale-out
~~~~~~~~~~~~~~~~~~~~~~

-  On Prism Central / Burger / Service / Karbon

-  Click on your cluster / Nodes on the left side / + Add Worker and add
   1 additional node (please don’t do more than one to keep resources
   for everyone) / Create. The system will deploy and add additional
   worker nodes. You can go back in a couple of minutes to see the
   additional worker added (around 5 minutes).

Check Karbon ElasticSearch / Kibana logging stack
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  On Prism Central / Burger / Service / Karbon

-  Click on your cluster / Add-on / Logging

-  Go to Discover in Kibana. Under Create Index Pattern, type \* and
   click Next Step / Select @timestamp time Filter / Create Index
   Pattern

-  Go again to Discover, and select the index \* on the top. You’ll see
   all logs of the K8S deployment.

Check object metrics 
~~~~~~~~~~~~~~~~~~~~

-  On Prism Central / Burger / Service / Object

-  Click on your cluster / performance on the left side. You’ll see the
   full performance overview (change to Last 1 hour to have a better
   view)

-  Click Buckets on the left side / nextcloud / performance. You’ll see
   the performance of the specific bucket

Clone the MariaDB Database
~~~~~~~~~~~~~~~~~~~~~~~~~~

-  On Era Dashboard click on the top menu / Times Machines

-  Click on your time machine / Action

-  Click Create Clone of MariaDB Instance from Time Machine

-  Select a specific Point in Time. It will deploy a clone with the
   content of the database at a specific time / next

-  Create a New Server

   -  Database Server VM Name : mariadb-*yourinitial*-0\ **2**

   -  Compute Profile : DEFAULT_OOB_COMPUTE

   -  Network Profile : MariaNW

   -  SSH KEY : Select Text, and copy paste the following string (it’s a
      one line text!)

.. code-block:: Bash
    
    ssh-rsa AAAAB3NzaC1yc2EAAAABJQAAAQEAiC8r6cLFLn/c/iR8TKXQhN20wUQwua8DSZM7rpGwuxbgLSSznW/hEVIogx3UoRamU3lIDsD8QKLBiHg29xc/PvR/Ro5Fxvhih3XOQTC14cEwPvgXgMHgPBJ5Vw+bW3a8HVM3S4dsaCsYAkDeHJmXP4G7HN4vrqc3fjb1UYV3iUe8AcheKzD7sG8MSjFBPc7WVI0I47Ly/eKVxVp0csE0fUH6IogUMqA1zp/C/uziAG1vZO6Td2S/FW70OKnCnnNRN8+e7BNlrIuy/0fLsKjUeNEgr8iuFFDoPA23vaPzcZR3hbsICOw7yoFbAsL+z+Mc6O74Nj7bT6WX3rVgMCFFYQ== 

-  Click next, enter the following information

   -  Name : nextcloud_02

   -  Database Parameter Profile : DEFAULT_MARIADB_PARAMS

   -  New ROOT password : nx2Tech123!

-  Check schedule data Refresh. When selecting this option, the system
   will periodically retrieve the data from the source database, and
   publish it to the clone you are deploying. Very useful for DEV and
   Test platform.

Manage your Kubernetes Cluster with LENS IDE
--------------------------------------------

On your jumphost, download and install the LENS Kubernetes IDE located
at this address: https://k8slens.dev/ Choose the current
Lens-Setup-x.x.x.exe

To graphically manage the K8S cluster, the LENS IDE can be used.

-  Open the LENS IDEN

-  Click File / Add Cluster

-  Select the previously downloaded kube configuration file and keep the
   default value / Add cluster(s)

-  You’ll now see all K8S ressources graphically.

For Instructors
===============

Link to Lab
-----------

https://docs.google.com/document/d/1Olu6xKqkbwPKplyKfI6ijDtlHb9XIZIIr15kXNCBLLU/edit#heading=h.yptqwv4kn3ni

.. |image0| image:: media/image3.png
   :width: 6.5in
   :height: 3.34722in
.. |image1| image:: media/image1.png
   :width: 3.08333in
   :height: 3.88542in
.. |image2| image:: media/image4.png
   :width: 3.39583in
   :height: 3.69792in
.. |image3| image:: media/image6.png
   :width: 0.44792in
   :height: 0.40625in
.. |image4| image:: media/image7.png
   :width: 0.29801in
   :height: 0.24503in
.. |image5| image:: media/image7.png
   :width: 0.29801in
   :height: 0.24503in
.. |image6| image:: media/image5.png
   :width: 1.97917in
   :height: 0.375in
.. |image7| image:: media/image2.png
   :width: 0.21875in
   :height: 0.29167in
.. |image8| image:: media/image8.png
   :width: 0.53125in
   :height: 0.45833in
