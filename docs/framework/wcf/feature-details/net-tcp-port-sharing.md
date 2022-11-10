---
title: "Net.TCP Port Sharing"
description: Learn about a TCP-based protocol for high-performance communication and the service that enables ports to be shared across multiple user processes in WCF.
ms.date: "03/30/2017"
helpviewer_keywords: 
  - "port activation [WCF]"
  - "port sharing [WCF]"
ms.assetid: f13692ee-a179-4439-ae72-50db9534eded
---
# Net.TCP Port Sharing

Windows Communication Foundation (WCF) provides a new TCP-based network protocol (net.tcp://) for high-performance communication. WCF also introduces a new system component, the Net.TCP Port Sharing Service that enables net.tcp ports to be shared across multiple user processes.  
  
## Background and Motivation  

 When the TCP/IP protocol was first introduced, only a small number of application protocols made use of it. TCP/IP used port numbers to differentiate between applications by assigning a unique 16-bit port number to each application protocol. For example, HTTP traffic today is standardized to use TCP port 80, SMTP uses TCP port 25, and FTP uses TCP ports 20 and 21. Other applications using TCP as a transport can choose another available port number, either by convention or through formal standardization.  
  
 Using port numbers to distinguish between applications had security problems. Firewalls are generally configured to block TCP traffic on all ports except for a few well-known entry points, so deploying an application that uses a non-standard port is often complicated or even impossible due to the presence of corporate and personal firewalls. Applications that can communicate over standard, well-known ports that are already permitted, reduce the external attack surface. Many network applications make use of the HTTP protocol because most firewalls are configured by default to allow traffic on TCP port 80.  
  
 The HTTP.SYS model in which traffic for many different HTTP applications is multiplexed onto a single TCP port has become standard on the Windows platform. This provides a common point of control for firewall administrators while allowing application developers to minimize the deployment cost of building new applications that can make use of the network.  
  
 The ability to share ports across multiple HTTP applications has long been a feature of Internet Information Services (IIS). However, it was only with the introduction of HTTP.SYS (the kernel-mode HTTP protocol listener) with IIS 6.0 that this infrastructure was fully generalized. In effect, HTTP.SYS allows arbitrary user processes to share the TCP ports dedicated to HTTP traffic. This capability allows many HTTP applications to coexist on the same physical machine in separate, isolated processes while sharing the network infrastructure required to send and receive traffic over TCP port 80. The Net.TCP Port Sharing Service enables the same type of port sharing for net.tcp applications.  
  
## Port Sharing Architecture  

 The Port Sharing architecture in WCF has three main components:  
  
- A Worker Process: Any process communicating over net.tcp:// using shared ports.  
  
- The WCF TCP transport: Implements the net.tcp:// protocol.  
  
- The Net.TCP Port Sharing Service: Allows many worker processes to share the same TCP port.  
  
 The Net.TCP Port Sharing Service is a user-mode Windows service that accepts net.tcp:// connections on behalf of the worker processes that connect through it. When a socket connection arrives, the port sharing service inspects the incoming message stream to obtain its destination address. Based on this address, the port sharing service can route the data stream to the application that ultimately processes it.  
  
 When a WCF service that uses net.tcp:// port sharing opens, the WCF TCP transport infrastructure does not directly open a TCP socket in the application process. Instead, the transport infrastructure registers the service’s base address Uniform Resource Identifier (URI) with the Net.TCP Port Sharing Service and waits for the port sharing service to listen for messages on its behalf.  The port sharing service dispatches messages addressed to the application service as they arrive.  
  
## Installing Port Sharing  

 The Net.TCP Port Sharing Service is available on all operating systems that support WinFX, but the service is not enabled by default. As a security precaution, an administrator must manually enable the Net.TCP Port Sharing Service prior to first use. The Net.TCP Port Sharing Service exposes configuration options that allow you to manipulate several characteristics of the network sockets owned by the port sharing service. For more information, see [How to: Enable the Net.TCP Port Sharing Service](how-to-enable-the-net-tcp-port-sharing-service.md).  
  
## Using Net.tcp Port Sharing in an Application  

 The easiest way to use net.tcp:// port sharing in your WCF application is to expose a service using the <xref:System.ServiceModel.NetTcpBinding> and then to enable Net.TCP Port Sharing Service using the <xref:System.ServiceModel.NetTcpBinding.PortSharingEnabled%2A> property.  
  
 For more information about how to do this, see [How to: Configure a WCF Service to Use Port Sharing](how-to-configure-a-wcf-service-to-use-port-sharing.md).  
  
## Security Implications of Port Sharing  

 Although the Net.TCP Port Sharing Service provides a layer of processing between applications and the network, applications that use port sharing should still be secured as if they were directly listening on the network. Specifically, applications that use port sharing should evaluate the process privileges under which they run. Consider running your application using the built-in Network Service account, which runs with the minimal set of process privileges required for network communication.  

## Known issue with update KB5018542
This update will prevent the port sharing service from running.  Below is the 2 step process to re-enable it.

Paste below into a .reg file and import to registry editor, then reboot.

Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetTcpPortSharing]
"Description"="@%systemroot%\\Microsoft.NET\\Framework64\\v4.0.30319\\ServiceModelInstallRC.dll,-8200"
"DisplayName"="Net.Tcp Port Sharing Service"
"ErrorControl"=dword:00000001
"FailureActions"=hex:84,03,00,00,00,00,00,00,00,00,00,00,03,00,00,00,14,00,00,\
  00,01,00,00,00,c0,d4,01,00,01,00,00,00,e0,93,04,00,00,00,00,00,00,00,00,00
"ImagePath"=hex(2):25,00,73,00,79,00,73,00,74,00,65,00,6d,00,72,00,6f,00,6f,00,\
  74,00,25,00,5c,00,4d,00,69,00,63,00,72,00,6f,00,73,00,6f,00,66,00,74,00,2e,\
  00,4e,00,45,00,54,00,5c,00,46,00,72,00,61,00,6d,00,65,00,77,00,6f,00,72,00,\
  6b,00,36,00,34,00,5c,00,76,00,34,00,2e,00,30,00,2e,00,33,00,30,00,33,00,31,\
  00,39,00,5c,00,53,00,4d,00,53,00,76,00,63,00,48,00,6f,00,73,00,74,00,2e,00,\
  65,00,78,00,65,00,00,00
"ObjectName"="NT AUTHORITY\\LocalService"
"RequiredPrivileges"=hex(7):53,00,65,00,43,00,72,00,65,00,61,00,74,00,65,00,47,\
  00,6c,00,6f,00,62,00,61,00,6c,00,50,00,72,00,69,00,76,00,69,00,6c,00,65,00,\
  67,00,65,00,00,00,00,00
"ServiceSidType"=dword:00000003
"Start"=dword:00000002
"Type"=dword:00000020

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NetTcpPortSharing\Security]
"Security"=hex:01,00,04,80,84,00,00,00,90,00,00,00,00,00,00,00,14,00,00,00,02,\
  00,70,00,05,00,00,00,00,00,14,00,fd,01,02,00,01,01,00,00,00,00,00,05,12,00,\
  00,00,00,00,18,00,ff,01,0f,00,01,02,00,00,00,00,00,05,20,00,00,00,20,02,00,\
  00,00,00,14,00,8d,01,02,00,01,01,00,00,00,00,00,05,0b,00,00,00,00,00,14,00,\
  14,00,00,00,01,01,00,00,00,00,00,05,04,00,00,00,00,00,14,00,14,00,00,00,01,\
  01,00,00,00,00,00,05,06,00,00,00,01,01,00,00,00,00,00,05,12,00,00,00,01,01,\
  00,00,00,00,00,05,12,00,00,00


Go to Windows Features and uncheck .NET Framework 4.7 and click okay.  This will uninstall .net and do a reboot
Go back to Windows Features and re-check .NET Framework 4.7.  Make sure the TCP Port Sharing in the WCF Services submenu is checked.  Click okay and reboot to reinstall .net4.7
After the computer comes back up the net.tcp port sharing service should be running okay.

## See also

- [Configuring the Net.TCP Port Sharing Service](configuring-the-net-tcp-port-sharing-service.md)
- [Hosting](hosting.md)
- [How to: Configure a WCF Service to Use Port Sharing](how-to-configure-a-wcf-service-to-use-port-sharing.md)
- [How to: Enable the Net.TCP Port Sharing Service](how-to-enable-the-net-tcp-port-sharing-service.md)
