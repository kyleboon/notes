## Overview  
  
In 2021 I continued being one of the most externally visible members of the team. I've worked across every team Snowfire/Keystone parters with and have an excellent working relationship with all the stakeholders. I contributed sgnificantly to Keystone accomplishing all of our goals for the year. In 2022 I am looking for new challanges and to expand my influence both internally and externally.   
  
## Organizational Influence  
* I was the technical mentor for Rifath Hussain for his TLP rotation from October 2020 - April 2021  
* I volunteer to do on-campus intern and TLP interviews. This often takes up several days a month during peak interview times in October/November and again in the Spring.  
* The Item Node Relationship Node to Node Serviceable Item data is core to every team in the middle mile supply chain. Over the summer they were having trouble with data inconsistencies in Kafka and I reviewed the code and gave feedback that helped them fix the root causes of their issues. Feedback  
* I'm very active in a lot of technical Slack channels including #java, #kotlin, #kafka-dev  
https://target.slack.com/archives/C83AVSG79/p1642519531039100  
https://target.slack.com/archives/CJWK0RN1J/p1642711716063200  
https://target.slack.com/archives/CJWK0RN1J/p1635527252017800  
https://target.slack.com/archives/C02LC0XBK35/p1642000307000400  
* Presented my talk Ditch Your Framework at the 2021 Opensource North conference  
  
## Technical Achievements  
  
My biggest achievement in 2021 was the design and initial implementation of Bedrock. In 2021 Keystone's biggest goal was to scale to 75% of RDC replenished item-locations. This represented about a 10x jump in activations over 2020. The biggest blocker for that goal was the speed of the node-si API. Bedrock was aimed at moving to a new node-si kafka topic as well as encapsulating all logic about replenishment dates (including beginning of season and realignment type information) into a single place. I proposed the plan and did the initial design and implementation of Bedrock. Bedrock enabled us to not only to get to 75% of RDC replenished item-locations, but to 100%. (Not all are being used in production, but are producing values for 100% of item-locations and future activations are a configuration change and won't require additional keystone development)  
This architecture diagram was originally created by me, but has been updated by others as the architecture has changed. https://confluence.target.com/display/KEYSTONE/Bedrock  
This work took up a significant portion of Spring and Summer 2021, then paused while INR and Node-Si fixed some issues, and finished up in the fall.  
  
Highlights by month:  
* 186 Pull requests merged across all organizations in 2021  
* January, February and March were focused on pipeline expansion. There was significant work to scale from our late 2020 activation numbers to our 2021 goals, and I did a lot of work with node-si, and the http loaders for eligibility (heimdall), demand curve coverage and delivery schedule. This work, particularly the Node-si work is what drove me to propose the bedrock design.  
* April, and May were focused on building out the initial bedrock implementation.  
* June - I spent doing validation of bedrock, which resulted in a lot of work with the node-si team and helping them overcome their issues with kafka. (see link above)  
* July - Debugged and fixed issues with offset committing and added new metrics to be able to monitor this easier  
* August - Took over leadtime kafka data source work when a coworker took a new job and got that into production  
* September - Focused on metrics and alerting improvements leading into peak season  
* October - Bedrock launch  
* November - Implemented changes required for 105 day horizon  
* December - Disaggreation deep dive  
  
I wanted to highlight the Unit Of Measure incident from August. I found and debugged a production support issue in which an API we used started returning incorrect data. I worked with Ashley, and we went from problem determination to production fix in a few hours. https://target.slack.com/archives/CVCR7CAGP/p1628521470008100 There was a longer term switch to a kafka topic to replace that http API, but our fix corrected all the data. I do not believe anyone but the two of us had the business context and deep knowledge of how Keystone works to turn this around as quickly as we did.