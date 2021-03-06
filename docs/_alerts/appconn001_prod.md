---
title: appconn#001 Microservice Unavailable
permalink: /alerts/appconn001_prod/
---

# When

Checking one of the following Slack channels:
All production alerts: `#appcon-ops-alerts`
{% for env_hash in site.data.envs.prod %}
{% assign env = env_hash[1] %}
        <p>{{ env.name }} alerts: <a href="{{ env.slackurl }}">
      {{ env.slacktitle }}
    </a></p>
{% endfor %}

You see `IBM Alert Notification` messages with content:

> Severity: Critical  
Where: [region] {a URL}  
What: Microservice not available - Playbook hip#001

For example:

![Screenshot of a typical alert](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/blob/master/wiki-images/TypicalAlert.jpg)

# Before you start
**Follow the steps in [Dealing with Incidents](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Dealing-with-incidents).**

# Do
1. Determine the deployment of App Connect that has issued the alert, US or UK. This is indicated by the first part of the **Where** text of the alert (in the above example, it's US South). 
2. Review the alert to determine what it is reporting.
3. [Investigate the App Connect logs](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Viewing-the-App-Connect-operational-logs) for the microservice in question.

### For home page outages (https://appconnect.ibmcloud.com/ or https://home.appconnect.ibmcloud.com/) 
Follow [hip#004 playbook](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/hip%23004-Marketing-page-slow-or-unavailable)

### For APIm service outages (Fusion)
Monitoring is in place for the APIm Controller and Gateway services in `us-south`, `eu-gb` and `au-syd`. Alerts will have a URL in the following form ([full list of service URLs](https://github.ibm.com/Cloud-Integration/firefly-api-personal-manager/blob/master/environment-configuration.md#health-monitor)):
- ```https://console.service.<region>.apiconnect.ibmcloud.com/fusion/controller```
- ```https://service.<region>.apiconnect.ibmcloud.com/gws/apigateway/health```

Contact the APIm team in slack ([#squad-api-fusion](https://ibmapim.slack.com/messages/C2VJEU96Y/) - first search for and join the API Connect workspace) if the alerts persist (a single ping fail is unlikely to be investigated by them). Identify you are working for `App Connect` and ask to verify if they have recorded the outage.

### For response time greater than 10s for single microservice
If there are slow responses for a specific microservice, [raise a HOI (not RCA)](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Managing-issues-on-the-Incidents-board#to-open-an-issue) and investigate the performance using the App Connect - System Latency dashboard and the application that caused the alert.

### Microservice unavailable or Return code error
####  --- What to do when you see a "Bluemix Blip" ...
* If you see large number (more than 10) microservices in **US-SOUTH** being reported as unavailable in within a couple of minutes of each other it is likely that this is an instance of a [problem we have been seeing for some time](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/issues/1769).  
    > - Raise a new HOI
    > - Put the start and end time of the alerts in the issue summary
    > - Do not capture logs/events or any further details
    > - Label with the `Service: App Connect`, `Region: US-South`, `external-ibm-cloud-platform` & `ping-test-fail`
    > - Move to `Recently Resolved` pipeline
    > - End
* For response times greater than 10 seconds. If there are slow responses reported across more than one microservice, [raise a HOI (not RCA)](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Managing-issues-on-the-Incidents-board#to-open-an-issue) and [check Bluemix status](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Identifying-the-correct-third-party) to determine if there have been recent network latency issues which might have impacted the service.
#### ...continue to work the issue as normal ---
We retry health monitor tests if there is a single 502 or 503 response before we raise a Critical alert.  

If there is an unavailable microservice reported [raise a HOI](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Managing-issues-on-the-Incidents-board#to-open-an-issue) and [check Bluemix status](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Identifying-the-correct-third-party) to determine if there have been any infrastructure issues which might have impacted the service.  

If Estado reported the outage make this clear in the HOI that downtime has been reported.

If there is an unavailable microservice reported for firefly-database-backup:
    - Check the [IBM Cloud status](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Identifying-the-correct-third-party) pages.  If there is a problem with `COMPONENT: Object Storage` in the region then this is very likely to be the root cause.  Record this in the HOI with a link to the outage notification.

If the issue persists, keep following the next sections in this playbook.

### Check for applications that aren't running on the App Connect dashboard 
1. Log in to [Bluemix](https://bluemix.net/), and change to the organisation `appconnect` and the space `production`('firefly-ops-commands should be the only service to be stopped.)  
2. Check whether the microservice listed in the alert is in running state (shown by a green dot). If it's not obvious which microservice it is, compare the route listed in the dashboard with the route mentioned in the alert.  
3. Find out whether Bluemix or any of our dependencies are down by loading [the Bluemix front page](https://console.ng.bluemix.net/) and using the information in the [third parties](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Identifying-the-correct-third-party) playbook to view their status page.
- If there are still issues, raise a ticket with Bluemix using the above playbook information.

### Check the `production-health` kibana dashboard to see if any apps did not respond to ping request 
1. Log on to [Grafana for Bluemix UK](https://metrics.eu-gb.bluemix.net/app/#/grafana4) or [Grafana for Bluemix US](https://metrics.ng.bluemix.net/app/#/grafana4) and switch org to `appconnect` and space to `production`.   
2. Click on the `Home` icon in the top left and search for dashboard `IBM App Connect - Dashboard` to see how many services are down and the service ping health, and dashboard `IBM App Connect - Health Monitor` to check the status of the health-monitor service.

Check for failed ping requests for the microservice listed in the alert.   
Are the ping failures still occuring - the problem may have been resolved and a ping request may now be successful for the microservice 

### Collecting diagnostic information
It may be useful to collect some status and information on the application that is unavailable.  

From your laptop using the **app.connect.ops** taskid
 - cf login -a {"https://api.eu-gb.bluemix.net" | "https://api.ng.bluemix.net"} 

    - To see a report on the application state, run `cf app {appname}`
    - To see a list of recent events (such as configuration changes or crashes), run `cf events {appname}`
    - To see the environment variable settings, run `cf env {appname}`
    - To see any logs that were printed to stdout (such as node errors), run `cf logs --recent {appname}`

You may find that multiple applications are unavailable.  In such cases, it may be useful to collect all of the information above from all applications.  You can then try to find patterns across the applications.  You can collect all of the information as follows:

- You will be creating a collection of files, so create a subdirectory and cd into it
- Run the following set of commands:

```bash
cf apps > apps.txt
for app in `grep -e started -e stopped apps.txt | cut -d' ' -f1`; do cf app $app > $app.app & done; wait
for app in `grep -e started -e stopped apps.txt | cut -d' ' -f1`; do cf events $app > $app.events & done; wait
for app in `grep -e started -e stopped apps.txt | cut -d' ' -f1`; do cf env $app > $app.env & done; wait
for app in `grep -e started -e stopped apps.txt | cut -d' ' -f1`; do cf logs --recent $app > $app.log & done; wait
```

### Restarting a microservice
If the microservice is still not responding to a ping then:
- Capture the recent CPU and memory usage metrics and any available logs for the microservice to provide to the squad for diagnosis and root cause investigation.
- **try restarting the microservice** by following the steps in playbook [Restart Apps in Production](https://github.ibm.com/Cloud-Integration/hip-ops-incidents/wiki/Restart-apps-in-production)  
   - If this fails then search for a playbook or a known issue that might be able to fix the issue.
   - Contact an expert in the area to try to resolve the issue.
   - Escalate as appropriate.

# Notes

## Maintainers 
Squad: Neptune
