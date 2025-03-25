# __*Splunk PowerUser - 10 Days Training Notes*__

Welcome to my 10 Days Splunk Power User Training Notes! 🚀 This repository contains structured notes, practical exercises, and key takeaways from my learning journey to master Splunk as a power user.

#__*📌 About This Repository*__

This repository serves as a personal reference and a resource for anyone looking to learn or revise Splunk fundamentals and advanced concepts. Each day's notes will cover essential topics with practical examples.

## 🚀 Splunk Training - Day 1 Topics

#__*Types of Dashboard*__

#__*Splunk Enterprise Components*__

#__*License Calculation*__
- Type of licenses
- How licensing works
- How data is measured

#__*License Pooling*__

#__*Splunk Forwarders*__
- Difference between Universal Forwarder and Heavy-Forwarder
- Why to use Heavy-Forwarder?

#__*Index Management*__
- Type of Index
- Buckets

#__*📁 Splunk Directory Structure*__

#__*⚙️ Job Management*__
- Managing long running jobs
- Search restriction through `authorize.conf`

---
## 🚀 Splunk Training - Day 2 Topics

#__*Difference between Real-time and Relative-time Search*__

#__*🔍 Type of Search Modes*__
- ⚡ Fast
- 🤖 Smart
- 🧐 Verbose

#__*Important Splunk Commands*__
- `| table` command
- `| rename` command
- `| stats` command - `count`, `sum`, `avg`, `list`, `values`
- `| fillnull` command
- `| eval` command
- `| eval` with `if-else` command
- `| eval` with `case` command
- `| sort` command
- `| top` command with `limit`
- `| rare` command with `limit`
- `| fields` command with `+/-`
- `| chart` command

#__*❓ Quiz*__
**Which query is most optimized?** 🤔

---

## 🚀 Splunk Training - Day 3 Topics

#__*| chart command*__
- Stack in chart
- ❓Difference between output of |chart and |stats outputs ?** 🤔
- Trellis in chart

#__*Single Value Visualization*__
- Single Value
- Radial Guage
- Marker Guage
- Filler Guage


#__*`| timechart` command*__
- use of `span`
- ❓In `|timechart` command, how to use X-axis as other timestamp field instead of default _time field ?** 🤔
- Trendlines

#__*`| geostats` command*__

#__*`| geom` command*__
-❓How to highlight countries inside a map, based on their count ?** 🤔
-❓How to convert kml file to kmz ?** 🤔

#__*Applications (App)/Add-ons*__

#__*SanKey Diagram*__

#__*`| addcoltotals` command*__
#__*`| addtotals` command*__
#__*`| rex` command*__

#__*Type of Searches*__

#__*`| makeresults` command*__

---

## 🚀 Splunk Training - Day 4 Topics

#__*knowledge objects*__

#__*Calculated Fields*__

#__*Tags*__

#__*EventType*__

#__*Lookups*__
- Type of Lookup files
- `|inputlookup` command
- `| lookup` command
- Ways to enrich data through lookup - `|join` or `lookup definitions`
- Automatic lookups
- Lookup Editor Application
- `|outputlookup` command  (💀!!!DANAGEROUS!!!)
- kvstore lookup

#__*Macro*__
- macro without argument
- macro with single argument
- macro with multiple arguments

---
## 🚀 Splunk Training - Day 5 Topics

#__*Data Models and Pivots*__
- Create Datamodel
- How to create `Auto-Extracted` field
- How to create `Calculated` field with `Eval-Expression`
- How to create `Lookup` field
- Acceleration

#__*Pivot*__
- Create a Pivot and save as Report or Dashboard-Panel
- `| datamodel` command

#__*Splunk Alerts*__
- Create and schedule an alert with action
- ❓Difference between 'webhook' and 'API' ?** 🤔

#__*Splunk Reports*__
- Create and schedule an alert with action

#__*Field Alias*__

#__*Field Expression*__
- `Regular` : Splunk will write expression
- `Delimiter` : symbol-wise splitting of event

#__*Workflow Action*__
- Two Action-Type : 
	- (1) link 
	- (2) search
---
## 🚀 Splunk Training - Day 6 Topics

#__*Universal Forwarder*__
- Download and Install UF
- Fixing `minimum free disk space (5000MB)` issue
- Enable Port 9997 on Enterprise and configure UF to communicate on 9997
- Check Connectivity at UF end through command
- Check Connectivity at Splunk Enterprise end through SPL
- Create custom log and forward its data to splunk
	- configuring `inputs.conf` in '../system/local' [NEVER in in '../system/defult']
	- configuring `outputs.conf` in '../system/local' [NEVER in in '../system/defult']
- Trobleshooting ways

---
## 🚀 Splunk Training - Day 7 Topics

#__*HEC (Http Event Collector)*__
- How to create HEC Token
- Push data through CURL command prompt using HEC token
- Push Multiple Events
- How to use HEC token if Splunk-cloud is hosted on AWS
- Indexer acknowledgement
- To Push a JSON data, define sourcetype as "_json"

#__*Apps/Add-on*__
- Install ServiceNow Add-on and Integrate with Splunk

#__*Forwarder Management*__

#__*Splunk Architecture in a Distributed environment*__

---	
## 🚀 Splunk Training - Day 8 Topics

#__*Event Breaking*__
- using Console "Source Type" Properties
- Event splitting on XML File
- ❓ Difference between LINE_BREAKER and MUST_BREAK_AFTER ? 🤔
- Event splitting on Random File 

#__*Timestamp Customization*__
- using Console "Source Type" Properties

#__*User and Roles*__
- Capabilities
- Roles
	- Native capabilities  and Inherited capabilities
	- Indexes
	- Restrictions
	- Resources
		- Role search job limit
		- User search job limit
		- Role search time window limit
		- Disk space limit
- Create User

#__*License Management*__

---	
## 🚀 Splunk Training - Day 9 Topics

#__*Dashboard*__
- `Classic` type
- `Studio` type
- Create a `Classic` Dashboard
- Optimize Process of a Dashboard :
	- using `Base search`
	- using `Saved search`
	- using `summary index`
		- Manual creation of `summary index`
		- Automated creation of `summary index`
- ❓ How to pass the value from One dashboard to other ? 🤔
- Create a `Studio` Dashboard

#__*Scripted Input in Splunk*__

---
## 🚀 Splunk Training - Day 10 Topics

#__*Forwarder Management*__
- Deployment Server
	- deploying dummy apps on clients
	- How to Connect DS with UFs
	- How create a ServerClass to deploy delivery app on Forwarders
	
#__*Index Time Field Extraction*__	
- Search-Time-Field-Extraction
- Index-Time-Field-Extraction
- working with `tranforms.conf` and `props.conf`


#__*Unix Integration*__
- Installing  `Splunk Add-on for UNIX and Linux`

- `| rest` command

#__*Folder Structure*__
---
📌 *Stay tuned for more!*


	

#__*📩 Contact*__

If you have any questions or suggestions, feel free to reach out via GitHub issues!
Happy Learning! 🎉
