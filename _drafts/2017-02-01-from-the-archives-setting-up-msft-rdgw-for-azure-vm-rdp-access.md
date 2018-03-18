---
layout: post
title:  "From the Archives - Setting up Msft RDGW for Azure VM RDP Access"
date:   2017-02-01 09:09:07 -0500
categories: windows, azure, vm managment, AD, security
---
At a previous job I stepped into managing virtual machines in Azure (Service Managment at the time) that were not joined to a domain.  RDP access was required and I began by helping out the team by responding to those requests and adding local admin user accounts and updating endpoint ACL's with the user's source IP address.  I wanted to be helpful, and the setting folks up was appreciated, it's just that, as you can probably tell where I'm headiing with this, it was a huge time suck.  Better said, it was `toil`.  As the team grew there were several reqeusts comming in each week due to home dynamic ip's changing and forgotten user names/passwords.  It simply did not scale.

There were a couple of problems to solve here in addition to providing access.  This was an opportunity to:
1.  Get rid of local accounts across several servers
2.  Scriptable account creation
3.  Reduce the number of ports exposed by removing RDP ports from server ACL's
4.  Central lize access logs
5.  Provide account self-management (password reset)
6.  Bonus: Add custom DNS records for configuration management pull servers!

The solution I implemented was, as part of our migration from Azure Service Management (ASM) to Azure Resource Management (ARM) vm's, to bite the bullet and set up an "on-prem" AD domain in a virtual network that could be easily peered with vm's running in different resource groups regardless of which data center (this was a valuable lesson in subnetting which I'll describe in a future post to the archives).  "On-prem" because this domain was not integrated initially for RBAC for Azure resources, and was only for login access to the vm's themselves. And "bite the bullet" because did I really want/need to manage AD domains (test and prod)?

Was I on the right track?  No way to know without trying!  I pushed forward with the idea.  First I began by using Terraform and its module capability to create my Azure resources I defined in a "region" module - meaning I might want to create these vm's in many regions.  Domain controllers would need to synchronize, and no need to serice login requests across a VPN when the time came to build out infrastructure in different Azure datacenters.  It wouldn't break the bank to have two small A0 sized vm's for reduncancy.  As part of the deployment, I knew I would be setting up my domain eg. "domain.azure" for replication between the 2 domain controllers the subnet, but I wasn't quite sure how to configure DNS to make this happen.  Next, I had some experience with RDGW but I wasn't really sure which Windows Server features would be required.  Keep reading for some of the lessons learned setting this up!

DNS
The key here is to set custom DNS server addresses for the virtual network to the private IP's of the two domain controllers.  This was easy enough to do within my terraform scripts.  A little bit tricker is setting the DNS server address of the alternative domain controller on the domain controler vm's network adapter.  Remember changing the DNS settings of the network adapter in the VM guest OS settings is never a good idea.

RDGW Config
Check out [Ryan Mangans IT Blog][add-rdgw-password-reset] for more info on how to set up the RDGW self management page to allow users to reset their expired passwords.

[add-rdgw-password-reset]: https://ryanmangansitblog.com/2013/03/11/add-password-reset-feature-to-remote-desktop-web-access-2012/
