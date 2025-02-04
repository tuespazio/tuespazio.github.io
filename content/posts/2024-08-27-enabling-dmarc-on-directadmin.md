---

layout: post  
title: Enabling DMarc on Directadmin  
date: 2024-08-27 13:58 -0600  
description:  
image:  
category: \[directadmin\]  
tags: \[directadmin,linux\]  
published: true  
sitemap: false

---

Adding a DMARC record  
DMARC is a system that adds some rules for DKIM and SPF to give remote servers a better idea of what your intention is for messages that fail those 2 systems. It doesn't really do anything other than clarifying the desired actions to be taken by remote servers, but should lower your spam score in some cases. Be sure that your SPF TXT value contains "-all" (not ~all) and that you have DKIM setup before proceeding.

To add a DMARC record when using the Evolution skin:

Navigate to "DNS Management"  
Click "Add Record"  
For "Record Type", select "TXT"  
TXT Record Type, select "DMARC"  
Enter your email into the "Aggregate Email (RUA)" field: spam-reports@domain.com  
Click "Add"  
For other skins, like the "Enhanced" skin, to add a DMARC record, go to your domain's DNS Management interface and add the following TXT record:

```
_dmarc TXT "v=DMARC1; p=none; sp=none; rua=mailto:spam-reports@domain.com"
Be sure to include the whole thing in "quotes".
```

The email address spam-reports@domain.com will sometimes be sent reported spam messages by larger providers. Not all services will email this address, but it's handy if a client using a large email provider clicks "this is spam" for a given message because you'll then be sent that message. Upon receipt, you can determine if someone at your domain is spamming or spoofing, or if the recipient incorrectly labeled the message as spam (at which point, you'd probably want to remove that recipient from your mail-out since they likely don't want it).

The rua=mailto:email portion is not mandatory if you don't want to get these emails.

You can change the values as desired, as per the DMARC specifications.

Google's DMARC guide can also be useful for constructing your DMARC record.

How to automate adding a new DMARC for new domains  
If you like the above setup, and you want to automate its addition to all new DNS zones, you can run the following:

```
cd /usr/local/directadmin/data/templates/custom
cp ../dns_txt.conf .
and then add the record you want to the bottom of the custom/dns_txt.conf by running this command:
```

echo '\_dmarc="v=DMARC1; p=none; sp=none; rua=mailto:spam-reports@|DOMAIN|"' >> dns\_txt.conf  
Keep in mind that this assumes that you have a 'spam-reports' email account created already.