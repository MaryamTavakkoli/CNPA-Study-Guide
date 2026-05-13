# Domain 6: Measuring Your Platform — Practice Questions

**Topics covered:** DORA metrics, developer productivity, platform efficiency measurement

---

**Question 1**

A platform team reports that their Deployment Frequency has increased from monthly to daily. According to DORA research, what does this indicate about the team's software delivery performance?

A. The team is taking on more risk because more frequent deployments mean more opportunities for failures  
B. The team has moved from a "Low" to an "Elite" performer category, indicating that they have reduced batch size, improved automation, and increased confidence in their delivery process  
C. The team should slow down to allow more time for manual testing between releases  
D. High deployment frequency is only meaningful for consumer-facing applications, not internal platforms  

**Answer: B**
DORA research (State of DevOps reports) classifies teams into performance tiers. Elite performers deploy on-demand or multiple times per day. Daily deployment frequency represents a major improvement from monthly, reflecting smaller batch sizes, faster feedback, and higher automation maturity.

---

**Question 2**

A platform team wants to measure the value it delivers to developer users. Which combination of metrics is MOST appropriate for demonstrating platform efficiency and developer experience improvements?

A. Cluster CPU utilisation and total cloud spend  
B. Number of platform team members and total lines of code in the platform  
C. Developer onboarding time, time-to-first-deployment for new services, platform adoption rate, and reduction in support tickets  
D. Number of Kubernetes namespaces and total number of deployed Pods  

**Answer: C**
Platform effectiveness is measured through developer-centric outcomes. Onboarding time, time-to-first-deployment, adoption rate, and support ticket volume are direct indicators of whether the platform is reducing cognitive load and enabling developer autonomy.

---

**Question 3**

According to the DORA four key metrics framework, which metric measures the percentage of deployments that result in a degraded service or require remediation?

A. Deployment Frequency  
B. Change Lead Time  
C. Mean Time to Recovery (MTTR)  
D. Change Failure Rate  

**Answer: D**
Change Failure Rate (CFR) is the proportion of deployments that cause production incidents requiring rollback or hotfix. It is a stability metric. Together with MTTR (a recovery metric), CFR measures the reliability dimension of software delivery performance.

---

**Question 4**

A platform team uses the SPACE framework to measure developer productivity. Which dimension of SPACE captures whether developers feel their tools enable them to do their best work without unnecessary friction?

A. Satisfaction and well-being  
B. Performance  
C. Activity  
D. Collaboration and communication  

**Answer: A**
The SPACE framework (Satisfaction, Performance, Activity, Communication/Collaboration, Efficiency) includes Satisfaction as a first-class dimension. Satisfaction measures developer sentiment about their tools, workflows, and environment — crucial for identifying hidden toil that activity metrics alone would miss.

