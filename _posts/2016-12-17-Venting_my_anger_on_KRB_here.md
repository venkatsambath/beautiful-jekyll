---
layout: post
title: How Kerberos works, fails, irritates, makes you impatient
---

We will proceed in increasing level of irritators.

Short intro:

Authentication service (AS). The AS issues TGTs to client for admission to the ticket-granting service. Before network clients can get tickets for services, each client must get an initial TGT from the AS.

Ticket-granting service (TGS). The TGS issues tickets admission to other services in the TGS’s domain or to the ticket-granting service of a trusted domain. When a client wants access to a service, it must contact the ticket-granting service in the service’s account domain, present a TGT, and ask for a ticket. If the client does not have a TGT valid for admission to that TGS, it must get one through a referral process that begins at the TGS in the user account’s KDC and ends at the TGS in the service account’s KDC.

===

Lets see an example:
===================

What goes behind a tgt request?




[root@nightly59-1 ~]# kinit venkat
[19513] 1481954811.39817: Getting initial credentials for venkat@GCE.CLOUDERA.COM
[19513] 1481954811.40349: Sending request (217 bytes) to GCE.CLOUDERA.COM
[19513] 1481954811.40439: Resolving hostname nightly59-1.gce.cloudera.com
[19513] 1481954811.40722: Sending initial UDP request to dgram 172.31.112.11:88
[19513] 1481954811.41314: Received answer from dgram 172.31.112.11:88
[19513] 1481954811.41340: Response was not from master KDC
[19513] 1481954811.41376: Processing preauth types: 19
[19513] 1481954811.41397: Selected etype info: etype des3-cbc-sha1, salt "(null)", params ""
[19513] 1481954811.41402: Produced preauth for next request: (empty)
[19513] 1481954811.41407: Salt derived from principal: GCE.CLOUDERA.COMvenkat
[19513] 1481954811.41410: Getting AS key, salt "GCE.CLOUDERA.COMvenkat", params ""
Password for venkat@GCE.CLOUDERA.COM: 
[19513] 1481954815.570163: AS key obtained from gak_fct: des3-cbc-sha1/7923
[19513] 1481954815.570258: Decrypted AS reply; session key is: des3-cbc-sha1/E06E
[19513] 1481954815.570282: FAST negotiation: available
[19513] 1481954815.570327: Initializing FILE:/tmp/krb5cc_0 with default princ venkat@GCE.CLOUDERA.COM
[19513] 1481954815.570542: Removing venkat@GCE.CLOUDERA.COM -> krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM from FILE:/tmp/krb5cc_0
[19513] 1481954815.570549: Storing venkat@GCE.CLOUDERA.COM -> krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM in FILE:/tmp/krb5cc_0
[19513] 1481954815.570617: Storing config in FILE:/tmp/krb5cc_0 for krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM: fast_avail: yes
[19513] 1481954815.570639: Removing venkat@GCE.CLOUDERA.COM -> krb5_ccache_conf_data/fast_avail/krbtgt\/GCE.CLOUDERA.COM\@GCE.CLOUDERA.COM@X-CACHECONF: from FILE:/tmp/krb5cc_0
[19513] 1481954815.570645: Storing venkat@GCE.CLOUDERA.COM -> krb5_ccache_conf_data/fast_avail/krbtgt\/GCE.CLOUDERA.COM\@GCE.CLOUDERA.COM@X-CACHECONF: in FILE:/tmp/krb5cc_0

=====

What goes behind a service ticket request?

[root@nightly59-1 ~]# kvno impala/nightly59-1.gce.cloudera.com
[20198] 1481954931.902491: Getting credentials venkat@GCE.CLOUDERA.COM -> impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM using ccache FILE:/tmp/krb5cc_0
[20198] 1481954931.902686: Retrieving venkat@GCE.CLOUDERA.COM -> impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM from FILE:/tmp/krb5cc_0 with result: -1765328243/Matching credential not found
[20198] 1481954931.902743: Retrieving venkat@GCE.CLOUDERA.COM -> krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM from FILE:/tmp/krb5cc_0 with result: 0/Success
[20198] 1481954931.902750: Found cached TGT for service realm: venkat@GCE.CLOUDERA.COM -> krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM
[20198] 1481954931.902760: Requesting tickets for impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM, referrals on
[20198] 1481954931.902807: Generated subkey for TGS request: des3-cbc-sha1/64A6
[20198] 1481954931.902816: etypes requested in TGS request: aes256-cts, aes128-cts, des3-cbc-sha1, rc4-hmac, des-cbc-crc, des, des-cbc-md4
[20198] 1481954931.903121: Sending request (767 bytes) to GCE.CLOUDERA.COM
[20198] 1481954931.903190: Resolving hostname nightly59-1.gce.cloudera.com
[20198] 1481954931.903419: Sending initial UDP request to dgram 172.31.112.11:88
[20198] 1481954931.904121: Received answer from dgram 172.31.112.11:88
[20198] 1481954931.904143: Response was not from master KDC
[20198] 1481954931.904221: TGS reply is for venkat@GCE.CLOUDERA.COM -> impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM with session key des3-cbc-sha1/E2F5
[20198] 1481954931.904240: TGS request result: 0/Success
[20198] 1481954931.904244: Received creds for desired service impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM
[20198] 1481954931.904250: Removing venkat@GCE.CLOUDERA.COM -> impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM from FILE:/tmp/krb5cc_0
[20198] 1481954931.904255: Storing venkat@GCE.CLOUDERA.COM -> impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM in FILE:/tmp/krb5cc_0
impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM: kvno = 2

===
How does a tgt looks like?

klist -e
Ticket cache: FILE:/tmp/krb5cc_0
Default principal: venkat@GCE.CLOUDERA.COM

Valid starting     Expires            Service principal
12/16/16 22:06:51  12/16/16 22:21:51  krbtgt/GCE.CLOUDERA.COM@GCE.CLOUDERA.COM
	renew until 12/16/16 22:36:51, Etype (skey, tkt): des3-cbc-sha1, des3-cbc-sha1 

12/16/16 22:08:51  12/16/16 22:21:51  impala/nightly59-1.gce.cloudera.com@GCE.CLOUDERA.COM
	renew until 12/16/16 22:36:51, Etype (skey, tkt): des3-cbc-sha1, des3-cbc-sha1 

===
How my user principal looks?

kadmin.local:  getprinc venkat
Principal: venkat@GCE.CLOUDERA.COM
Expiration date: [never]
Last password change: Fri Dec 16 22:06:15 PST 2016
Password expiration date: [none]
Maximum ticket life: 30 days 00:00:00
Maximum renewable life: 31 days 00:00:00
Last modified: Fri Dec 16 22:06:15 PST 2016 (HTTP/admin@GCE.CLOUDERA.COM)
Last successful authentication: [never]
Last failed authentication: [never]
Failed password attempts: 0
Number of keys: 6
Key: vno 1, des3-cbc-sha1, no salt
Key: vno 1, arcfour-hmac, no salt
Key: vno 1, des-hmac-sha1, no salt
Key: vno 1, des-cbc-md5, no salt
Key: vno 1, des-cbc-crc, Version 4
Key: vno 1, des-cbc-crc, AFS version 3
MKey: vno 1
Attributes:
Policy: [none]

=====

kdc.conf
supported_enctypes = des3-hmac-sha1:normal arcfour-hmac:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal des-cbc-crc:v4 des-cbc-crc:afs3

=====

Though my KDC has 7 etype support, why my TGT was encrypted with des3-cbc-sha1 key?

Ans: Since des3-cbc-sha1 comes first in the order - it is assigned first  

aes256-cts-hmac-sha1-96 aes128-cts-hmac-sha1-96 des3-cbc-sha1 arcfour-hmac-md5 camellia256-cts-cmac camellia128-cts-cmac des-cbc-crc des-cbc-md5 des-cbc-md4


