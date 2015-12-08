# Convention for Semantic Hostnames (CSH) [![GitHub Stars](https://img.shields.io/github/stars/semantic-hostnames/semantic-hostnames.svg)][stargazers]

## Overview

The rules for creating hostnames are covered in [RFC 952][rfc952], [RFC 1123][rfc1123], and further amended in [RFC 2181][rfc2181].

From the [Wikipedia][wikipedia_hostname] article:

> Hostnames are human-readable nicknames that correspond to the address of a device connected to a network. They are used by various naming systems, e.g., Network Information Service (NIS), Domain Name System (DNS), Server Message Block (SMB), and the meaning of hostname will vary according to the naming system used. A hostname meaningful to a Microsoft NetBIOS workgroup may be an invalid Internet hostname. When presented with a hostname without any context, it is usually safe to assume that the network is the Internet and the hostname's naming system is the DNS.
 
However, the latest one of these (RFC 2181) was written in 1997, and the Internet has moved on.

## The Problem of Treating Hostnames as Domain Names

When the hostname spec was first created, it assumed that one server on the Internet did something. There was one server named www.yourcompany.com, and it alone handled your HTTP traffic. It sat in your comms room, because you hosted your own server, and it had one IP address.

In the modern world of [immutable infrastructure][immutable_infrastructure], and treating servers as [cattle rather than pets][cattle_pets], this schema is less useful. What is your `www` server? Your load balancer? Your WAF? Your [CloudFlare][cloudflare] account?

There is also the question of _domain_ itself. Are Cloud servers on [Amazon AWS][aws] your domain? Or [Digital Ocean][digital_ocean]? How many domains resolve to those servers? Do the names of those servers mean anything to you?


## Making Hostnames Have Modern Meaning 

The concept of semantic hostnames is inspired by a [blog post][sd_blog] by [Server Density][server_density], although the naming convention in the article again follows the concept of _domain_. Domain names can, and should, be decoupled from hostnames. Domains may change, but your hostname should be for the life of the device.

The aim is to have a naming convention that is:

- Easily searched. Whether just throwing names in a spreadsheet, grepping logs or files, or full blown Elastisearch, having a convention that exposes key information about the host from the name alone is useful.
- Still useful at the terminal. As most terminals will chop to the first period, it's useful to have meaningful data at the command line.

### Pragmatism at the Terminal

Let's take a couple of "classic" hostnames for a server: firstly, a digital agency running `uat.some-client.myagency.com`, and an in-house team creating a live microservice hosted on `users.mycoolstartup.com`. What do these look like at the terminal when you're trying to fix them?

The digital agency name (assuming using [oh-my-zsh][ohmyzsh] with "bira" theme):

```bash
╭─root@uat  ~/semantic-hostnames
╰─$
```
The cool startup microservice:
```bash
╭─root@users  ~/semantic-hostnames
╰─$
```

While the above tell me _something_ about the environment I'm logged into, I would prefer to know more.

## Are names still relevant?

As more and more developers move to cloud infrastructure, asset management is moving away from individual hosts. Autodiscovery, tags and regions are becoming more important than naming any one server. Nevertheless, individual servers are still important at the SME level, to institutions migrating from bare tin, or spread across providers. Human legibility is still important; being able to identify a host from a casual glance at a screen is useful, even in a world where you may abstract that away.

While no longer the canonical source it once was, naming something - or rather, branding cattle in your herds - is still useful, and co-exists with autodiscovery. While everybody wants to move to a point where they never have to SSH into a box to figure out what's going on, the reality for many is not yet inline with the ideal. To use an oft-quoted adage, _"In theory, theory and practice are the same. In practice, they are not"_.

### An example semantic hostname

`acme-pipe-uat-nginx01.cen6-rs-vm-lon-uk`

Breaking this down into components:

`[group]-[tier]-[environment]-[role][increment].[os]-[hosting provider]-[device type]-[locale]-[region]`

What can we tell about the above server just from the name?

Well, assuming the context of, say, a digital agency, this server is for our `Acme` client. If we only do in-house projects, it's our `Acme` project or microservice. It's a non-production pipeline server based on the `pipe` flag, and is used for `UAT`. It's primary role is as a Web server using `nginx`, and it's the first server. It also is running CentOS 6 as based on the `cen6` part, hosted by [Rackspace][rackspace]. It's virtual, i.e. not bare metal, based on `vm`, and it is located in their London data centre in the United Kingdom.

At the terminal, we'll see:

```bash
╭─root@acme-pipe-uat-nginx01  ~/semantic-hostnames
╰─$
```
Now I know I'm not going to accidentally restart the live production nginx instance, or working on the wrong project.

## Creating a semantic hostname

A _semantic hostname_ is broken down into two or more labels, separated by periods.

- Business value
- Infrastructure
- Optional: Image data (AMI info, etc)

The third label, _Image Data_, is not covered here as it is very specific to individual implementations.

The RFC states the entire name must be no more than 253 octets, and each period-separated label should be no more than 63 octets. Underscores are not allowed. Each part of the label is hyphenated to split into meaningful slugs.

### First label: Business Value

#####`group`
**Required**. This may be a project, a company name, a client, or a domain, though I recommend against the latter. 

#####`tier`
_Optional_. This allows you to have a sweeping boolean between production and your pipeline. You may have many different non-production environments: UAT, QA, OAT, beta, demo and so forth. Rather than create searches on all of these, a flag of `pipeline` or `pipe` can help. If you prefer to skip this, you can use inverse searching with `env != 'prod'` 

#####`environment`
**Required**. The environment itself, for example `prod` for production, `qa`, `uat` and so forth.

#####`role`
**Required**. The _role_ of the server in your architecture. If you have multiple roles for the server (i.e. a server with Apache and MySQL running on it), then choose one as primary or use a generic role, i.e. `web`.

#####`increment`
**Required** The increment of the above role. Default is `01`, but if you are into the thousands of servers, feel free to start with higher bounds such as `00001` (although you probably already have your own naming convention by that scale).


### Second label: Infrastructure

#####`os`
**Required**. The operating system of the device. You should use the major version release number as the last part of this, i.e. `rhel6` for Red Hat 6, `win10` for Windows, `osx10` for a Macbook, etc.

#####`hosting provider`
**Required**. If you host your own servers, this should be the team name, i.e. `ops`. You need to know who to ring if there's a problem.

#####`device type`
_Optional_, but recommended. Short code for the device: `sp` for smartphone, `vm` or `vrt` for virtual machine, `lb` for load balancer, and so on.


#####`locale`
_Optional_.  A region-specific slug, i.e. `lon` for London, UK.

#####`region`
**Required**. The region the device is in. If it's a mobile device, the country of issue.


## Semantic hostnames and FQDNs

Semantic hostnames and [Fully Qualified Domain Names (FQDNs)][wikipedia_fqdn] don't have to be mutually exclusive. As previously mentioned, decoupling hostnames from domains is advantageous, but if you wish to make your hostname both _semantic_ and _fully qualified_, then they can both co-exist:

`acme-pipe-uat-nginx01.cen6-rs-vm-lon-uk.somedomain.com`

## Specification by example

A host running as part of a DB cluster, (second machine of 5) in UAT for the Wibble project. The server is running Red Hat 6 and is hosted by AWS in their US East 1 region.

`wibble-pipe-uat-dbcluster02.rhel6-aws-vm-us-east-1`

The same machine, but the team does not wish to use the `tier` slug:

`wibble-uat-dbcluster02.rhel6-aws-vm-us-east-1`


A single live host running a microservice for Single Sign On for Cool Startup. The machine is running Ubuntu 14x on Digital Ocean in their London environment:

`cs-prod-sso01.ub14-do-vm-lon-uk`


## The main thing to remember

Be consistent. The convention above may not be ideal for you, but if you choose to change order of label parts, make sure you do it throughout. Don't have some servers following one naming convention and another following another unless you can guarantee you will never need to look through any sort of list or search containing _both_.

## Automating Naming

I've written an [Ansible role][ansible_semantic_hostname] that makes the process of naming cattle relatively painless. A lot of it should be fire and forget.

## The Author

The Convention for Semantic Hostnames was created by [shrikeh][shrikeh] after working for years trying to normalise naming of a herd of different servers with different operating systems for different clients, projects, and roles. Tell me what you think of it on [Twitter][twitter].

[wikipedia_hostname]: https://en.wikipedia.org/wiki/Hostname "Article regarding hostnames on Wikipedia"
[rfc952]: https://www.ietf.org/rfc/rfc952.txt "IETF RFC 952" 
[rfc1123]: https://www.ietf.org/rfc/rfc1123.txt "IETF RFC 1123"
[rfc2181]: https://www.ietf.org/rfc/rfc2181.txt "IETF RFC 2181"
[immutable_infrastructure]: https://blog.codeship.com/immutable-infrastructure/ "Blog post by Codeship on the concept of Immutable Infrastructure"
[cattle_pets]: http://www.lauradhamilton.com/servers-pets-versus-cattle 'Excellent summary of the "Pets vs Cattle" concept'
[cloudflare]: https://www.cloudflare.com/ "CloudFlare home page"
[sd_blog]: https://blog.serverdensity.com/picking-server-hostnames/ "Server Density blog on hostnames"
[server_density]: https://www.serverdensity.com/ "Server Density home page"
[ohmyzsh]: http://ohmyz.sh/ "Oh my ZSH home page"
[rackspace]: http://www.rackspace.com/ "Rackspace home page"
[aws]: https://aws.amazon.com/ "Amazon AWS homepage"
[digital_ocean]: https://www.digitalocean.com "Digital Ocean homepage"
[wikipedia_fqdn]: https://en.wikipedia.org/wiki/Fully_qualified_domain_name "FQDN description on Wikipedia"
[shrikeh]: https://github.com/shrikeh "My GitHub profile"
[twitter]: https://twitter.com/barney_hanlon "My Twitter page"
[stargazers]: https://github.com/semantic-hostnames/semantic-hostnames/stargazers "Stargazers on GitHub"

[ansible_semantic_hostname]: https://github.com/shrikeh/ansible-semantic-hostname "Semantic hostname role Galaxy"