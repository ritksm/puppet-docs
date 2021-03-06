---
layout: default
title: "PE 3.2 » Deploying PE » External CA"
subtitle: "Using an External Certificate Authority with Puppet Enterprise"
canonical: "/pe/latest/external_ca.html"
---

PE uses its own certificate authority (CA) to generate and verify certificates and security credentials (private and public keys and certificate revocation lists) for the various elements that make up a PE deployment. However, you may already have your own CA in place and wish to use it instead of PE's. This doc will familiarize you with the certs signed by the PE certificate authority and then detail the procedures for replacing those certs and security credentials.    

> ###*Before You Begin* 
> Setting up an external certificate authority (CA) to use with PE is beyond the scope of this document; in fact, this writing assumes that you already have some knowledge of CA and security credential creation and have the ability to set up your own external CA. This document will lead you through the certs and security credentials you'll need to replace in PE. However, before beginning, we recommend you familiarize yourself with the following docs:
>
>- [SSL Configuration: External CA Support](http://docs.puppetlabs.com/puppet/latest/reference/config_ssl_external_ca.html) provides guidance on establshing an external CA that will play nice will Puppet (and therefore PE).
> 
>- [ActiveMQ TLS](http://docs.puppetlabs.com/mcollective/reference/integration/activemq_ssl.html) explains MCollective’s security layer.

##About the PE Certificate Authority and Security Credentials

After installing PE, you can run `puppet cert list --all` to inspect the following inventory of certificates signed using PE's built-in CA: 

- puppet master (and any agent nodes)
- pe-internal-broker 
- pe-internal-dashboard
- pe-internal-mcolllective-servers
- pe-internal-peadmin-mcollective-client
- pe-internal-puppet-console-mcollective-client

You will also need to replace the CA cert and security credentials in the PuppetDB SSL directory.

These certs will all need to be replaced with new certs signed by your external CA. 

###Locating the PE Master Certificate and Security Credentials

You will need to replace the CA and security credential files on the puppet master. (We've included instructions for adding agent nodes at the end of the doc.)

To determine the proper locations for the CA and security credential files, run the following commands with `puppet master`:

- **Certificate**: `puppet master --configprint hostcert`
- **Private key**: `puppet master --configprint hostprivkey`
- **Public key**: `puppet master --configprint hostpubkey`
- **Certificate Revocation List**: `puppet master --configprint hostcrl` 
- **Local copy of the CA's certificate**: `puppet master --configprint localcacert` 

>**Tip**: You will also need to [create a cert and security credentials for any agent nodes](#adding-agent-nodes-using-your-external-ca) using the same CA as you used for the puppet master. We've included instructions at the end of the doc.

###Locating the PE Console Certificate and Security Credentials

The following files, located on the puppet master, or on the console server in a split install, need to be replaced:

- `pe-internal-dashboard.ca_cert.pem` (replace with your CA cert)
- `pe-internal-dashboard.private_key.pem`
- `pe-internal-dashboard.ca_crl.pem` (replace with your CA CRL)
- `pe-internal-dashboard.public_key.pem`
- `pe-internal-dashboard.cert.pem`

These files are stored at `/opt/puppet/share/puppet-dashboard/certs/`.

###Locating the PuppetDB Certificate and Security Credentials

The following files, located on the puppet master, or on the PuppetDB server in a split install, need to be replaced: 

- `/etc/puppetlabs/puppetdb/ssl/ca.pem` (replace with your CA cert)
- `/etc/puppetlabs/puppetdb/ssl/private.pem` (replace with your master's private key)
- `/etc/puppetlabs/puppetdb/ssl/public.pem` (replace with your master's cert)

###Locating PE MCollective Certificates and Security Credentials

The following files, located on the puppet master, need to be replaced:

- `pe-internal-broker.pem` (controls the ActiveMQ server)
- `pe-internal-mcollective-servers.pem`
- `pe-internal-peadmin-mcollective-client.pem`
- `pe-internal-puppet-console-mcollective-client.pem`

These certs and security credentials are generated by the puppetlabs-pe\_mcollective module as part of the PE installation process. You will need to replace the cert, private key, and public key for each of these. These files are stored at `/etc/puppetlabs/puppet/ssl/certs`, `/etc/puppetlabs/puppet/ssl/private_keys`, and `/etc/puppetlabs/puppet/ssl/public_keys`. 

##Replacing the PE Certificate Authority and Security Credentials

>**Important**: For ease of use, we recommend naming *ALL* of your certificate and security credential files exactly the same way they are named by PE and replace them as such on the puppet master; for example, use the `cp` command to overwrite the file contents of the certs generated by the PE CA. This will ensure that PE will recognize the files names and not overwrite any files when you perform puppet runs. In addition, this will prevent you from needing to touch various config files, and thus, limit the chances of problems arising.  
>
> The remainder of this doc assumes you will be using identical files names. 

We recommend that once you've set up your external CA and security credentials, you first replace the files for PE master/agent nodes and the PE console, then replace the files for PuppetDB, and then replace the PE MCollective files. Remember, naming the new certs and security credentials exactly as they're named by PE will ensure the easiest path to success. 

Here is a list of the things you'll do:

1. Install PE <link>. 
2. Choose a certificate authority option.
3. Use your external CA to generate new certificates and security credentials to replace all existing certificates and security credentials.
4. Replace the PE master, PE agent, and PE console certs and security credentials.
5. Replace the PE PuppetDB certs and security credentials.
6. Replace the PE MCollective certs and security credentials.


###Replace the PE master and PE Console Certificates and Security Credentials

1. Refer to [Locating the PE Master Certificate and Security Credentials](#locating-the-pe-master-certificate-and-security-credentials) and copy your new certs and security credentials to the relevant locations.
2. Refer to [Locating pe-internal-dashboard Certificate and Security Credentials](#locating-pe-internal-dashboard-certificate-and-security-credentials) and copy your new certs and security credentials to the relevant locations.
3. On the puppet master, navigate to `/etc/puppetlabs/puppet/puppet.conf`, and in the `[master]` stanza, add `ca=false`.
4. Run `service pe-httpd restart`. 
5. Continue to [Replace PuppetDB Certificates and Security Credentials](#replace-the-puppetdb-certificates-and-security-credentials).   

###Replace the PuppetDB Certificates and Security Credentials

1. Refer to [Locating the PuppetDB Certificate and Security Credentials](#locating-the-puppetdb-certificate-and-security-credentials) and replace the files. 

   >**WARNING**: Be sure you replace `/etc/puppetlabs/puppetdb/ssl/public.pem` with your master's cert---not your master's public key.

2. Run `service pe-puppetdb restart`.
3. Run puppet.

After running puppet, you should be able to access the console, and view your new certificate in your browser. However, live management will not work; you can access that part of the console, but it won't be able to find the master node.

Now you will need to replace the MCollective certificates and security credentials.

###Replace the MCollective Certificates and Security Credentials 

1. On the puppet master, ensure that you replaced the CA cert (`ca.pem`) at `/etc/puppetlabs/puppet/ssl/certs/`. (**Tip**: If you didn't, the above procedures wouldn't have worked.)
2. Refer to [Locating PE MCollective Certificates and Security Credentials](#locating-pe-mcollective-certificates-and-security-credentials) and replace the cert, private key, and public key for each of these.
3. On the puppet master, navigate to `/etc/puppetlabs/activemq/`.
4. Remove the following two files: `broker.ts` and `broker.ks`.
5. Run `service pe-activemq restart`.
6. Run `service pe-mcollective restart`.
7. Run puppet. 

After puppet runs, the presence of `/etc/puppetlabs/puppet/ssl/certs/ca.pem`, will regenerate the `broker.ts` and `broker.ks` files that are needed for the ActiveMQ truststore and keystore. 

You should now see the master node in live management and be able to perform puppet runs and other live management functions using the console.

##Adding Agent Nodes Using Your External CA 

1. Using the same external CA you used for the puppet master, create a cert and private key for your agent node. 
2. Using `puppet agent` as the prefix, run the same commands in [Locating the PE Master Certificate and Security Credentials](#locating-the-pe-master-certificate-and-security-credentials) to replace the cert and private key for your agent node.
3. Copy the CRL and CA certs from the master node to the correct locations on the agent node. (Again, use `puppet agent` as the prefix.)
4. Run puppet. 

You should now be able to access your agent node from the console; however, you may not yet see your node in live management (controlled by MCollective). In such cases, you can often force the connection by waiting a few minutes and then running `puppet agent -t`. Once the connection has been made, the output in your terminal will show that the MCollective server has picked up the node. 

If you still don't see your agent node in live management, use NTP to verify that time is in sync across your PE deployment. (You should *always* do this anyway.)
 
