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
📌 *Stay tuned for more!*


	

#__*📩 Contact*__

If you have any questions or suggestions, feel free to reach out via GitHub issues!
Happy Learning! 🎉
