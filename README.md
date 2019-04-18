# Active Directory Splunk Dashboard Template
This repo contians a baseline dashboard that once updated for your envionrment should produce interesting metrics for your domain(s).
* Login load by domain controller
* Aggregate Auths
* Kerberos Encryption Types in use
* NTLM usage/data
* Firewall Info

![Sample AD Dashboard](https://github.com/nextinstall/splunk-dashboards/blob/master/ADSampleDashboard.png)

## Getting Started
Copy the ad-splunk-dashboard.xml file and edit the search/replace the following variables to reflect your enviornment.


### Prerequisites
Advanced Auditing policy enabled. Use Center for Internet Security or Microsoft's Security Compliance Toolkit baseline audit recomendations at a minimul to generate relevant events.
If you aren't collecting the data, this dashboard will be useless.

For Firewall logging you MUST enable Windows Firewall logging to collect the data. You MUST also tell Splunk to vaccum up the c:\windows\system32\Logfiles\firewall\*


## Deployment
Copy and augment the .xml file to suit your enviornment. Create a new Splunk dashboard and edit source.
Pate the contents of the .xml in their entierity. 


## Built With
* [Splunk](http://www.splunk.com) - SIEM Used


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