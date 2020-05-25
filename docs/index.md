Betterprotect is a Postfix Management System. It includes a Log Parser, the ability to whitelist/blacklist addresses and LDAP integration. 
 
## Log parser
Betterprotect directly parses Postfix log messages, ensuring you'll always see the latest status

## Policy
Betterprotect stores all of your data inside a policy.
This policy is installed on the Postfix server.
This central configuration approach enables you to easily keep multiple servers in sync.

## Blacklist/Whitelist
Betterprotect includes a policy service, which enables you to whitelist or blacklist specific IPs or networks.
It's also possible to combine them with senders
You can for example deny messages from the IP "192.168.1.1" with a sender address of "spam@contoso.com", but accept messages from the same IP with a sender address of "nonspam@contoso.com"

## Transport
Easily store all of your mail transport rules in Betterprotect.

## Relay domains
Add your relay domains in Betterprotect and push them to your mail servers.

## LDAP integration
Betterprotect not only supports LDAP sign in, but can also pull recipient address from multiple LDAP directories. 
These recipients are then feed to Postfix, so only messages for people that actually exist within your organisation are accepted.