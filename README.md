# Active Directory Splunk Dashboard Template
This repo contians a baseline dashboard that once updated for your envionrment should produce interesting metrics for your domain(s).
* Login load by domain controller
* Aggregate Auths
* Kerberos Encryption Types in use
* NTLM usage/data
* Firewall Info

![Sample AD Dashboard](https://github.com/nextinstall/splunk-dashboards/blob/master/ADSampleDashboard.png)

## Getting Started
Download the ad-splunk-dashboard.xml file and edit/replace the following variables to reflect your enviornment:

index=*INDEXNAME*

host=*HOST_NAME_SCHEMA*

*DOMAINANME*

IE:

If my index is AD-Systems-Index:

`index=AD-Systems-Index`

If my DC's have names like AD-DC01, AD-DC02, AD-DC03...:

`host=AD-DC0*`

If my NETBIOS domain name is CORP and my FQDN is: CORP.CONGLOMO.ORG:

replace `DOMAINNAME` with `CORP`

Some event analysis `EventCode=476*` might be required to ensure events are properly filtered such that: 

`Account_Domain=CORP` 

OR

`Account_Domain=CORP.CONGLOMO.ORG`


### Prerequisites
Advanced Auditing policy enabled. Use Center for Internet Security or Microsoft's Security Compliance Toolkit baseline audit recomendations at a minimul to generate relevant events.
If you aren't collecting the data, this dashboard will be useless.

For Firewall logging you MUST enable Windows Firewall logging to collect the data. You MUST also tell Splunk to vaccum up the c:\windows\system32\Logfiles\firewall\*
The Firewall panel expects you to be be tagging your firewall logs with sourcetype=WindowsFirewall.  If you use another sourcetype label you should augment the XML.

## Deployment
Copy and augment the .xml file to suit your enviornment. Create a new Splunk dashboard and edit source.
Pate the contents of the .xml in their entierity. 


## Specific Search Queries Used
**1 hour auth load by DC**

```
index=*INDEXNAME*) (host=*HOST_NAME_SCHEME*) sourcetype="WinEventLog:Security" EventCode=4624 | stats count by host
```

**Total Authentications / minute**

```
index=*INDEXNAME*) (host=*HOST_NAME_SCHEME*) sourcetype="WinEventLog:Security" EventCode=4624  | timechart count
```

**Kerberos Encryption in use 0x17 = rc4-hmac | 0x12 = aes256**

```
index=*INDEXNAME* Account_Domain=DOMAINNAME host=* EventCode=476* Keywords="Audit Success" Ticket_Encryption_Type="*"|top limit=20 Ticket_Encryption_Type |eval "KERBEROS Encryption Type"=case(Ticket_Encryption_Type="0x17","rc4-hmac",Ticket_Encryption_Type="0x12","aes256-cts-hmac-sha1-96")
```

**NTLMv1 1hr Auth Volume**

```
(index=*INDEXNAME*) (host=*HOST_NAME_SCHEME*) Authentication_Package=NTLM Package_Name__NTLM_only_="NTLM V1" Account_Domain=DOMAINNAME | timechart count by host
```

**Top 20 NTLMv1 Authenticated User Accounts**

```
(index=*INDEXNAME*) Account_Domain=DOMAINNAME Authentication_Package=NTLM Package_Name__NTLM_only_="NTLM V1" Account_Name!=*$ | top limit=20 Account_Name
```

**Top 10 NTLMv1 Auths per Server**

```
(index=*INDEXNAME*) host=* Authentication_Package=NTLM Package_Name__NTLM_only_="NTLM V1"| timechart count by host limit=10
```

**Blocked Auths - IF Restrict NTLM**

```
(index=*INDEXNAME*) EventCode=8004 | stats count by User_name,Workstation_name,host
```


**DOMAINANAME - INBOUND - DC Connections - DNS=53, KRB=88, LDAP=389, LDAPS=636**

```
index=*INDEXNAME*) host=*HOST_NAME_SCHEME* sourcetype="WindowsFirewall" | eval fields= split(_raw, " ")|eval date=mvindex(fields,0) |eval time=mvindex(fields,1)|eval action=mvindex(fields,2)|eval protocol=mvindex(fields,3)|eval src_ip=mvindex(fields,4)|eval dst_ip=mvindex(fields,5)|eval src_port=mvindex(fields,6)|eval dst_port=mvindex(fields,7)|eval size=mvindex(fields,8)|eval tcpflags=mvindex(fields,9)|eval tcpsyn=mvindex(fields,10)|eval tcpack=mvindex(fields,11)|eval tcpwin=mvindex(fields,12)|eval icmptype=mvindex(fields,13)|eval icmpcode=mvindex(fields,14)|eval info=mvindex(fields,15)|eval path=mvindex(fields,16)|search path=RECEIVE AND (dst_port!=-) |stats count by host,dst_port,src_ip
```

####OPTIONAL:

ADD something like this as an AND into the query to filter your DC->DC Communications if possible/desired!

```
(src_ip!=10.xx.yy.* AND src_ip!=127.0.0.1 AND src_ip!=10.xx.zz.*)
```

If you have certain Splunk apps you may not need such an awful field extraction and may be properly tagging these logs on ingestion. If so that's great, remove my field evals and just count by host, ds_port, src_ip, or filter on those fields. Lucky you.

**LAPS Access by User**

```
(index=*INDEXNAME*) (EventCode=4662 OR EventCode=4624)  [|search ( index=*INDEXNAME*) (EventCode=4662 Account_Name!="*$" "ms-Mcs-AdmPwd"  NOT (Account_Name=exemptServiceAccount)) | table Logon_ID | dedup Logon_ID] | table _time,EventCode,Account_Name,Logon_ID,Object_Name,Source_Network_Address,Workstation_Name | eval Logon_ID=if(mvcount(Logon_ID)&gt;1,mvindex(Logon_ID,1),Logon_ID) | eventstats dc(Object_Name) AS Number_Accounts_Retrieved values(Object_Name) AS Accounts_Retrieved BY Logon_ID | where(EventCode=4624) | where(Number_Accounts_Retrieved&gt;0)| sort -_time
```
          

## Contributing
This is a base template, but share your panels and I'll integrate (and drop credit) if there is universal value. 

## Versioning
v1.0 - Inital Commit!

## Authors

* **Tom Gregory** - *Initial work* - [NextInstall](https://github.com/NextInstall)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments
* Harvard University Information Security & the Harvard AD Engineering Team - for paying me to do this!*

## Powered by
* [Splunk](http://www.splunk.com) - SIEM Used
