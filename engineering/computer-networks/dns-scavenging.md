[Computer networks](/engineering/computer-networks)
# DNS scavenging

**Scavenging** (also used interchangeably with **aging**) is a mechanism of removing old and unused records from DNS (zone) and in general works for dynamic records.
It's used for larger computer networks to automate possible mess and hostnames resolution problems from stale records.

DNS scavening works on timestamps of hostnames recorded in it. Intial timestamp gets recorded when client adds dynamic record in DNS. Then it gets updated automatically by clients in 24h intervals (but, AFAIK it's automatical for Windows hosts but so much for Linux ones).
Static records (in comparison to dynamic ones) don't get scavenged because their timestamps are set to 0.

From my current experience as a developer scavenging can become an issue for Linux hosts, that join the network and register themselves as dynamic DNS record on creation (because of their Terraform/other provisioner definitons etc.) but never get refreshed/updated, so they fall out of DNS and become unreachable.
