# **Generating EJScreen NPL Facility Proximity**

EJScreen uses Apache Hadoop Pig scripts to generate Superfund National Priority List (NPL) facility proximity. The Pig scripts were developed using Esri's [GIS Toolkit for Hadoop](https://esri.github.io/gis-tools-for-hadoop/) toolkit. It was run in an AWS EMR cluster environment. The source data came from EPA's Office of Land and Emergency Management (OLEM). The proximity process involves Pre-Hadoop processing, running Hadoop Pig scripts, and Post-Hadoop processing. The end results are Census block-group based proximity scores.

**Source:**

The source data is from the Superfund Enterprise Management System (SEMS) Public Access Database ([https://cumulis.epa.gov/supercpad/CurSites/srchsites.cfm](https://cumulis.epa.gov/supercpad/CurSites/srchsites.cfm)). Pulled from all "active" on February 8, 2023 – contains final (FinalNPL\_020823.xlsx) and proposed (ProposedNPL\_020823.xlsx).

**Pre-Hadoop Processing:**

- Merge final and proposed NPL together.
- Import spreadsheet to table (NPL\_020823 in geodatabase NPL\_Work.gdb).
- Download the NPL data from EPA Envirofacts.
- Join the two sets of NPL data based on EPA ID to obtain latitude and longitude coordinates for each NPL site. Note that the EPA ID used is the Program System ID (PGM\_SYS\_ID in the Envirofacts table).
- Drop records outside the 50 US states, DC and PR, to create table NPL\_020823\_forHadoop.
- Export table to NPL\_020823\_forHadoop.csv with these columns: EPA\_ID, LATITUDE, LONGITUDE, CWEIGHT; note that CWEIGHT = 1 for all records.

**AWS Hadoop Processing:**

- Start an AWS EMR cluster.
- Rather than doing 52 separate state runs, combine the states into 5 state groupings.
- Run Step 1 for each of 5 state groups to generate weighted distances facility-block pairs. See **NPL\_US01\_proximity\_Step1.txt** for Pig script example.
- Run Step 2 for each of 5 state groups to generate block group summary for each subgroup. See **NPL\_US01\_proximity\_Step2.txt** for Pig script example.
- Use Athena to create BG-level results tables and download as csv files, for example, BG\_Scores\_01.csv to OutputfromHadoop folder.
- Repeat all steps for each state group.

**Post-Hadoop Processing:**

- Combine all BG score text files in OutputfromHadoop folder into one file (NPL\_BG\_Scores\_US.csv).
- Open NPL_BG_Scores.csv in a text editor to capitalize the header and remove all other header rows. 
- Import US csv file to Excel; make sure BLKGRP is text.
- Port US Excel file into geodatabase table (NPL\_Work.gdb/NPL\_BG\_Scores\_US).
- Rename columns to STCNTRBG and BG\_SCORE, and name table NPL\_BG\_Scores\_Final.
- Provide datasets for testing. First create NPLProximity\_Testing.gdb.
- Include US\_BG\_Scores\_Final, and NPL\_020823\_forEJ tables.
- Add US\_NPLProx\_BG with BG shapes and BG\_SCORE, set NULL Scores to 0.

**EPA Disclaimer**

The United States Environmental Protection Agency (EPA) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. EPA has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by EPA. The EPA seal and logo shall not be used in any manner to imply endorsement of any commercial product or activity by EPA or the United States Government.
