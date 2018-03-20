PI System data ingress using OMF 
================================

You use OSIsoft Message Format (OMF) to achieve high-throughput asynchronous data ingress into the PI System. 
The following terms and references might be useful for understanding the information in this and subsequent topics: 

* A producer of OMF messages intended for the PI System is called an *OMF application instance*. 
* Every producer must be registered with the PI System to be able to ingress OMF data. For informatin about the 
  registration process, see *PI Data Collection Manager* user manual. 
* Producers are categorized by OMF application types. A Type serves as a namespace for unique OMF type definitions (see *OMF 1.0
  specification*). A type also translates into PI AF Element Templates (see *PI System Explorer user manual*). 
* OMF messages are divided into three categories (see *OMF 1.0 specification*), which perform several different actions in the PI System.
  The sequence of the messages, and whether they are required to be sent by a producer restart, are important to understand 
  (see Quick Start section). 

Introduction 
------------

Before creating your application, it helps to review and understand each of the following points:

* Understand your data and how to the data should be stored in the PI System back end. Also, understand how to write 
  OMF messages, so that appropriate AF templates, element trees (with required attributes), and associated PI points,
  are created in PI AF and PI Data Historian, and how they are updated with your timeseries data. 
* Start your code development by configuring the required environment, registering your development application instance with 
  the DCM/Relay (to obtain authorization to feed data into PI System), and write and send OMF messages to the 
  Relay's Ingress HTTP(S) REST endpoint. 
* Know how to make development changes to your OMF types, instances, and links. Know what must be manually cleaned up, 
  and how to troubleshoot your code. 

Understanding your data 
-----------------------

a. Identify your assets. 

   *  Physical – Physical assests represent things such as plants, sites, equipment, I/O devices, and so on. Assets may 
      have static attributes (which will remain immutable), or may have values that change and will not be recorded in 
      the data historian (such as a serial I/O device). 
   *  Logical – Logical assets represent things such as data streams (representing collections of values), which 
      should be sent to the Relay ingress as a single transaction; that is, all values of a given stream should 
      be sent in a single update; no one single value can be skipped. Data for each of these points in the stream 
      is recorded into the data historian with an appropriate timestamp. 
b. Identify hierarchical relationships between your assets and data streams. 

   *  Structures of physical assets - For example, you might have an asset at the top-most level, which contains a 
      collection of equipment, each of which has a collection of I/O devices. An example of this might be a vehicle 
      as a top-level asset with attached child asset engine and attached children's wheels. 
   *  Data stream assets, which can be attached to any of your physical assets. For example, a stream of two 
      values consisting of longitude and latitude can be attached to your vehicle asset, as well as a stream of two 
      values such as RPM and pressure of your vehicle's engine. 


Identify PI System's "reference model" of your data 
---------------------------------------------------

Your system's reference model allows you to categorize the physical and real-world aspects 
of your system. The reference model should model your system from a technical, engineering perspective, which can be understood
technical personel. It is not necessary to make the AF model and structure 
understandable to each and every person in your organization. Keep the structure simple! If you are adopting or adapting 
OSIsoft PI System, you should know the tools that are provided, such as AF Schema Mapper, 
(or what's its name???), which allows you to build different representations of your reference model for 
different business units of your organization. 

For more information about AF, see *PI AF Explorer* and *PI AF SDK* user manuals. 

a. Map your data's physical assets to AF elements. 
b. Link your data's physical assets together to create appropriate AF tree structure. 
c. Link your stream assets to appropriate AF elements to create dynamic element attributes, 
   which store their values in PI Data Historian. 

Write OMF messages to create your reference model and start feeding data into PI System 
---------------------------------------------------------------------------------------

For more information see *OMF 1.0* specification. 
 
a. Create OMF type definitions, which will represent your physical and data stream assets. 
   These will be sent to Relay ingress in OMF Type messages. 
   
   i.  Physical assets will be presented by OMF types with "classification": "static".
   ii. Data Stream assets will be presented by OMF types with "classification": "dynamic". 
   
b. Create OMF containers, which will represent instances of your data stream assets, which you will 
   later link to instances of your physical assets. These will be sent to Relay ingress in OMF Container messages. 
   
c. Create OMF physical (static) type instances. Later you will link them to each other and data stream 
   (dynamic) type instances (containers). These will be sent to Relay ingress in OMF Data messages. 
d. Create links between: 

   i.  Root node and static (physical) type instances. 
   ii. Static to static (parent/child) type instances. 
   iii. Static to dynamic (data stream) (parent/end leaf child) type instances. 
   
e. Update data stream (container) instances with time series values. These will be sent to Relay Ingress 
   in OMF Data messages, and update AF attributes with PI point references. 


Development Environment Cleanup 
-------------------------------

In OMF applications, type definitions and their representations in PI System, are immutable – you cannot 
make any changes to the properties of the type, which has been already sent to the Relay's ingress endpoint. 
The same is true for instances of these types (assets and containers), and linkage between them. Once you 
create concrete instances of these types and link them together by sending container and data messages to 
the Relay's ingress endpoint, you cannot redefine them. 

Once the OMF application has been developed and deployed, the only changed data that are expected by ingress 
is the values of your timeseries data – values sent to the container(s) in data messages. 

Only in error situations, when Relay loses its cache, type, asset and container, and their linkage information 
should be resent to the Relay. To recover from this, you can send this meta information every time your 
application restarts, having that NO changes were made to these definitions and instantiations. 
 
As a developer, you will have to deal with changes in the types, structures of your assets in AF, etc. 
To be able to perform such changes, you need to understand how to properly cleanup your development 
environment, and when cleanups are required. 
 
As a rule of thumb, you need to perform a cleanup: 

* Change was made to a type (basically – any change): 

  * You modified a name or description of the type or one of its properties 
  * You added, removed, renamed a property 
  * You changed a type of a property (i.e. from number to string, etc.) 
  
* Change was made to a container 

  * You redefined container typeid to another dynamic type 
  
* Change was made to a data (except of data values that you send to containerids): 

  * You redefined asset typeid to another static type 
  * You changed anything that you previously sent in the "__LINK" object 
 

*Cleanup:*

1. Relay's temporary cache location. 
   Stop the Relay process. By default, if not chosen during Relay setup, temporary data will be stored in
   C:\ProgramData\OSIsoft\Tau\Relay.ConnectorHost. Delete this folder. 
   Deleting this folder will remove all cache for all producers. 
   
2. PI System AF Database that you use to create your AF asset structure.
   In PI System Explorer, open Library, expand Templates/Element Templates. Delete all templates with 
   names starting with "OMF". 
   In PI System Explorer, open Elements, expand Elements root node. Delete all elements and their 
   children elements that has names of you OMF application instances registrations. 
   In PI System Explorer, check-in all your deletion changes. 
   
3. PI Data Archive PI points that were created once you sent container data values. 
   In PI System Management Tool, open Points/Point Builder. Search for PI tags that has names starting with 
   your OMF application instance registration. Delete all of them. 
 
Operation #1 is required always. 

Operation #2 is required if your application defines and links static types. 

Operation #3 is required if you previously sent data values to containers. 

