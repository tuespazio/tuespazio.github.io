---
categories:
- \[directadmin\]
date: "2024-08-27T00:00:00Z"
description: null
image: null
sitemap: false
tags: \[directadmin,linux\]
title: Enabling DKIM with DirectAdmin
---

DKIM is a means of authentication using cryptographic techniques designed to combat email spoofing. A DKIM-enabled mail server will generate a hash of the email contents that it wants to sign, use its private key to encrypt it, and sign the outbound message with it. Remote servers receiving email from the DKIM-enabled mail server will check that signature against DNS records added to the sending domain’s DNS zone and verify the signature. They do this by retrieving the public key from the domain’s DNS, using it to decrypt the encrypted hash, and then generating their own hash for comparison against the original hash to determine if they are equal (DKIM passes) or not (DKIM fails).

DirectAdmin gives the server administrator many options to enable DKIM in order to fit their needs, whether they want it enabled globally by default, on a per-domain basis, on a per-user basis, or just provide the option for users to enable it at will.

Important Considerations  
When enabling DKIM, it is important to understand that the DKIM DNS record is created and added to the local zone files. Thus, if any domains are using eternal DNS, you must copy these records over to the remote DNS zone file for those domains. Once configured on the local mail server for a domain, outgoing messages will be signed, so you’ll want to make sure that a public key exists in any remote zone files for a domain using remote DNS.

Enabling the Ability For the Users To Choose Whether To Enable DKIM  
To allow the users to choose whether or not to enable DKIM, you must set dkim to the value of 2 in the directadmin.conf and the rewrite Exim’s configuration files and recompile Exim for this. Log into the server via SSH as the root user and issue the following command to do this,

```
/usr/local/directadmin/directadmin set dkim 2 restart
cd /usr/local/directadmin/custombuild
./build update
./build exim
./build eximconf
Now, a user can enable DKIM with a click of a button in the DirectAdmin interface via User Level → E-Mail Accounts → Enable DKIM.
```

dkim directadmin  
Enabling DKIM Globablly By Default For All (Future and/or Existing Domains)  
For Future Domains  
If you’d prefer to have a DKIM record created by default upon domain creation, you would need to set to the dkim option in the directadmin.conf to a value of 1. To do this, run the following commands via SSH as the root user,

```
/usr/local/directadmin/directadmin set dkim 1 restart
cd /usr/local/directadmin/custombuild
./build update
./build exim
./build eximconf
Now, all future domains will have a DKIM record created by default.
```

For Existing Domains  
Enabling automatic DKIM record creation doesn’t affect existing domains, though. You have a few options for adding DKIM to the old domains once you’ve enabled dkim in the directadmin.conf. You can either do this for each domain one-by-one or for all existing domains at once.

To enable DKIM for all existing domains after setting dkim to 1 in the directadmin.conf, you can run the following command via SSH as the root user,

echo "action=rewrite&value=dkim" >> /usr/local/directadmin/data/task.queue; /usr/local/directadmin/dataskq  
To enable DKIM for only select domains one-by-one, use either the task queue or dkim\_create.sh script provided by DirectAdmin (replace DOMAIN.COM with the domain you’d like DKIM enabled for),

```
cd /usr/local/directadmin/scripts/dkim_create.sh DOMAIN.COM
Or
```

```
echo "action=rewrite&value=dkim&domain=DOMAIN.COM&dns=yes" >> /usr/local/directadmin/data/task.queue; /usr/local/directadmin/dataskq
Note: that both commands work the same way with the exception that you can have the DKIM written immediately with the dataskq verses within one minute with the dkim_create.sh script.
```

Disabling DKIM on a Per User Basis When Enabled or Permitted Globally  
This feature is designed so that DKIM is either enabled or permitted globally, but you can disable it on a per user basis. This requires either dkim=1 or dkim=2 set in the directadmin.conf. The setting of 1 will enable DKIM automatically for all domains under each user unless you specify otherwise in their user.conf files. The setting of 2 will allow them the ability to enable DKIM themselves unless otherwise specified in their user.conf file.

Note that setting dkim=0 in the directadmin.conf fully disables DKIM for the entire server and the user.conf files won’t be checked. Thus, if you want DKIM disabled globally by default with the option to enable for certain users/domains only, setting dkim=2 in the directadmin.conf is a better option.

When you create a user, you will have created them with a default domain. This domain will have DKIM created for the domain by default due to the globally enabled setting and the non-existence of a user.conf until after the account is created (no user.conf to edit and disable dkim). So, the DKIM for the default domain will need to be removed manually if you’d prefer it not have a DKIM record. Here are the steps to first remove the record for the default domain and then disable the automatic DKIM record creation for subsequent domains under that user.

To remove the records from the default domain, remove the x.\_domainkey TXT record from /var/named/DOMAIN.TLD.db and then remove the keys,

rm /etc/virtual/DOMAIN.COM/dkim.public.key  
rm /etc/virtual/DOMAIN.COM/dkim.private.key  
Now, edit the user.conf so that no subsequent domains created under the user have DKIM records enabled by setting dkim=0. The user.conf can be edited via SSH as the root user and the file is located here (where USERNAME represents the username of the user you are editing),

/usr/local/directadmin/data/users/USERNAME/user.conf  
This allows that particular user’s dkim setting to override the globally enabled dkim setting set in the directadmin.conf and thus prevents any DKIM records from being created for that user’s domains moving forward.

Restart DirectAdmin after making the changes,

service directadmin restart  
Disabling DKIM on a Per Domain Basis  
This is essentially the same feature as the per user feature, with the exception that you would edit the domain’s configuration file located in /usr/local/directadmin/data/users/USERNAME/domains/DOMAIN.TLD.conf, instead. You do not need to edit the user’s user.conf file for per domain control.

If a certain domain uses both a remote DNS and a remote mail server, you may want to disable DKIM for this one domain rather than all domains belonging to the user. This is where this feature is handy.

Restart DirectAdmin after making changes to the DOMAIN.TLD.conf file,

service directadmin restart  
Changing the Default Selector  
DirectAdmin uses x as the default selector. To change the selector, you will need to update the directadmin.conf with the desired selector. The following examples change the selector to default,

/usr/local/directadmin/directadmin set dkim\_selector default restart  
CustomBuild 2.0 rev 1824 will automatically swap the selector in the /etc/exim.dkim.conf. You will need to rebuild the Exim configuration like so,

/usr/local/directadmin/custombuild/build exim\_conf  
Confirm the selector was swapped in the /etc/exim.dkim.conf file,

\[root@host ~\]# grep -i selector /etc/exim.dkim.conf  
dkim\_selector = default  
Now, any newly created DKIM records will use your specified selector. Do note that you will need to remove and recreate the old DKIM records that use the old selector if you want them to use the new selector.