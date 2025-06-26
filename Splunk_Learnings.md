### Splunk Learning #1: Optimization of Azure Billing Dashboard Load Time — Reduced from 45 Seconds to 2 Seconds
 
(1) Implemented the command `| search "properties.subscriptionName"="$subscription_name$"` directly within the base search, rather than applying it individually within each panel.

(2) Previously, the base search included `index  + | fields + | search`. 
   This was modified to `index + | fields + | search + | stats by <required fields>` — where `<required fields>` refer to those used in panel-level statistics. 
   
(3)  Corresponding `minor adjustments were made to the SPL queries in each panel` to accommodate this change.

-----------------------------
