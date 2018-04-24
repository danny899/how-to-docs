# How to Setup a NAT Gateway and OpenVPN server on Cloud.

After few days of wondering around the web, I manage to setup a VPC network on the Cloud (AliCloud), with a dedicated ECS instance as the NAT gateway and OpenVPN server. Even many of the Cloud vendors (AWS, AliCloud, Azure, etc) nowadays provide NAT Gateway as a service, it is still more cost friendly to setup your own NAT gateway instance, especially when you combine both the NAT gateway service together with the OpenVPN server. I put up this document to help people who wants to setup their own NAT gateway instance without using NAT Gateway as a service.

In this document we used the following services:

<ul>
  <li>AliCloud ECS service - 1 x ecs.t5-lc1m1.small instance </li>
  <li>AliCloud Elastic IP service </li>
  <li>AliCloud VPC</li>
</ul>
  
We will go through both the NAT Gateway setup flow as well as the OpenVPN setup flow. The following diagram gives a high level picture of the NAT gateway setup. 

<img src="http://evertrue.github.io/images/2015-06-09-the-right-way-to-set-up-nat-in-ec2/how_vpc_routing_works.png" />


<h2>Setup a NAT gateway using AliCloud ECS instance</h2>

A NAT gateway routes traffic in/out to/from private network to the public network, and eventually connecting the private network with public network. To create a NAT gateway, first we need to create a VPC network on the cloud. 

<ul>
  <li>In AliCloud, go to ECS -> Virtual Private Cloud, choose "Create VPC" and assign an name for the VPC network.</li>
  <li>Assign a CIDR block "172.16.0.0/12" </li>
  <li>Create a VSwitch with Zone "Hong Kong Zone B" - CIDR block "172.16.0.0/24" </li>
</ul> 

Create a pair of subnats for each available zone, a private subnet and a public subnet.
