# commission_sdi
Web IDE project for data integration to SAP Commission using SDI

This repository explain how to import SDI project in your local workspace and create a working data integration solution using SDI. Once all the steps completed successfully, you will be able to build a Web IDE project which moves the data from remote data source (in this example flat file) to physical table in EXT schema. Followed by moving the data from EXT schema to TCMP schema. In this example we use CS_STAGEPARTICIPANT file for data integration. We will use Extract, Load and Transform pattern.

Following are the steps:-

1. Connenct the DP agent to the SDI server and enable the File Adapter
Follow these videos in case you are doing it for the first time. 
  
    https://www.youtube.com/watch?v=4anKSVRrHh0&t=379s
    
    https://www.youtube.com/watch?v=8j6lD7lP5B0
    
    Similarly in the linux system setup the following in the DP Agent file adapater configuration
    
    a.	Root directory
    
    b.	Format root directory
    
    c.	Enter access token
    
2. Download the CS_STAGEPARTICIPANT.txt in the your root directory (as setup in the privious step). This file contains the sample data  It acts as the source data in this example.

3. We also need to create the metadata file(.cfg) for the source data file to migrate the data. Execute the following command to create the metadata file. Replace the root directory information as setup in the previous step.
"createfileformat.bat -file <<rootdirectory>>\CS_STAGEPARTICIPANT.txt -cfgdir <<rootdirectory>> -firstRowAsColumnName true"
  
  you can find the "createfileformat" executable in the "AgentUtil" folder where the DP agent is installed for both Windows and Linux environment. In Linux use "createfileformat.sh".
  
  The resulting configuration file will look something like CS_STAGEPARTICIPANT.txt.cfg (this file is attached for just reference. Don't use this file as is becuase it wont work without modification in your environment).
 
 4. Create a remote source in the Hana DB as explained in the video 
    Please note - while creating the remote source in DB explorer in the Hana you need to provide SharePoint login. Use any dummy user and password (though it is not relevant, you can not save the configuration without putting dummy value)
    
5. Create a virtual table named - VT_CS_STAGEPARTICIPANT in EXT schema as shown in the video. Check if you can see the data using open data feature of the DB explorer.

6. Create a physical table in the EXT schema named CS_STAGEPARTICIPANT. It should have the same structure as of VT_CS_STAGEPARTICIPANT virtual table. You can use similar command like - CREATE TABLE  "EXT"."CS_STAGEPARTICIPANT" like  "EXT"."VT_CS_STAGEPARTICIPANT". The idea is to create flowgraph and first move the data from vitual table (created in previous step) to the physical table created in this step. 

7. Download the MigrateParticipant.zip in any folder. This is the Web IDE project which we will import in next step.

8. Log in the Web IDE. Select the "Workspace" in the Development area and right click to "Import File or Project". Use the browse button to navigate to the place where you kept MigrateParticipant.zip and select it. Keep all the default and select "Ok"

9. It should create a project in the Web IDE workspace. Right click on the project and go to "Project Setting". Select the space entry and save the the setting. There should be only one space listed in the drop down. Select that space. When we will deploy the project this space information will be used to create run time objects.

10. Expand the project. Select "HDBParticipant" node. Right click on it and select "Modeling Action" -> "Add Externa SAP Hana Service". We need to create this service to access table in EXT and TCMP schema. Web IDE project creates a HDI container to access Hana DB. It creates technical users to access the database(HDI container). These users only have access to the HDI container. We need two things 1. create service to access data from EXT and TCMP schema from Web IDE artifacts (which we are doing in this step). 2. create synonym for the EXT and TCMP tables as Web IDE artifacts (flowgraphs) can access these tables only via synonym and not directly. We will handle synonym in later steps.

10. In the popup dialog box for "Add External SAP Hana Service" set User Provided Service Name as "ServiceParticipant" and provide rest of the information.

11. Open the DB explorer and login to your tenant database. Open the SQL Console and execute following SQL statements

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."VT_CS_STAGEPARTICIPANT" TO MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "TCMP"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

AS mentioned earlier Web IDE project creates technical users, in this project it creates "MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO" user therefore in this step we have provided the access to this user to access EXT and TCMP tables. Please note - Web IDE flowgrahs can only access these table via synonyms which we will create in next step.

12. Now go back to the Web IDE project and expand the "HDBParticipant" node. Open "syn_participant.hdbsynonym" file by clicking on it. In the database column replace the name of the dabase from "SMF0001LAB" to your database name and save the file.

13. Select the file "syn_participant.hdbsynonym" and just build this selected file. In case everything goes fine it will create three synonyms to be used in this project else try to repeat the privious steps again.

14. Select the flowgraph "VT_to_EXT_Participant.hdbflowgraph" and build the selected file. Once executed this flowgraph will move the data from virtual table to physical table in EXT schema i.e. from "VT_CS_STAGEPARTICIPANT to CS_STAGEPARTICIPANT. 
Similarly select the flowgraph "EXT_to_TCMP_Participant.hdbflowgraph" and build the selected file. Once executed this flowgraph will move the data from table in EXT to TCMP schema tables.
If both the files are built successfully the project bulding is done. Congrats!

15. Now open "VT_to_EXT_Participant.hdbflowgraph" and in the UI click the "Execute" command. Once the flowgraph is executed the data will be moved from virtual table EXT.VT_CS_STAGEPARTICIPANT to physical table EXT.CS_STAGEPARTICIPANT.

16. Now open "EXT_to_TCMP_Participant.hdbflowgraph" and click on the "Execute" command. After the flowgraph execution is over the data will be moved from EXT.CS_STAGEPARTICIPANT table to TCMP.CS_STAGEPARTICIPANT table.

This completes the migration of the data from remote data source to the TCMP table.

Later we will see how to move the data from TCMP table to SAP Commission main table.
