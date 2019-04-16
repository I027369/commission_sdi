# commission_sdi
Web IDE project for data integration to SAP Commission using SDI

This repository explain how to import SDI project in your local workspace and create a working data integration solution using SDI. Once all the steps completed successfully, you will be able to build a Web IDE project which moves the data from remote data source (in this example flat file) to physical table in EXT schema. Followed by moving the data from EXT schema to TCMP schema. In this example we use CS_STAGEPARTICIPANT file for data integration.

Following are the steps:-

1. Connenct the DP agent to the SDI server and enable the File Adapter
Follow these videos in case you are doing it for the first time. 
  
    https://www.youtube.com/watch?v=4anKSVRrHh0&t=379s
    
    https://www.youtube.com/watch?v=8j6lD7lP5B0
    
    Similarly in the linux system setup the following in the DP Agent file adapater configuration
    a.	Root directory
    b.	Format root directory
    c.	Enter access token
    
2. Download the CS_STAGEPARTICIPANT.txt in the your root directory (as setup in the privious step)

3. Execute the following comand. Replace the root directory information as setup in the previous step.
"createfileformat.bat -file <<rootdirectory>>\CS_STAGEPARTICIPANT.txt -cfgdir <<rootdirectory>> -firstRowAsColumnName true"
  
  you can find the "createfileformat" executable in the "AgentUtil" folder where the DP agent is installed.
  
  The resulting configuration file will look something like CS_STAGEPARTICIPANT.txt.cfg (this file is attached for just reference. Don't use this file as is becuase it wont work without modification in your environment).
 
 4. Create a remote source in the Hana DB as explained in the video 
    Please note - while creating the remote source in DB explorer in the Hana you need to provide SharePoint login. Use any dummy user and password (though it is not relevant, you can not save the configuration without putting dummy value)
    
5. Create a virtual table named - VT_CS_STAGEPARTICIPANT in EXT schema. Check if you can see the data using open data feature of the DB explorer.

6. Create a physical table in the EXT schema named CS_STAGEPARTICIPANT. It shoudl have the same structure as of VT_CS_STAGEPARTICIPANT virtual table. You can use similar command like - CREATE TABLE  "EXT"."CS_STAGEPARTICIPANT" like  "EXT"."VT_CS_STAGEPARTICIPANT"

7. Download the MigrateParticipant.zip in any folder and keep it in a folder. 

8. Log in the Web IDE. Select the "Workspace" in the Development area and right click to Import File or Project. Use the browse button to navigate to the place where you kept MigrateParticipant.zip and select it. Keep all the default and select "Ok"

9. It should create a project in the Web IDE workspace. Right click on the project and go to "Project Setting". Select the space entry and save the the setting.

10. Expand the project. Select "HDBParticipant" node. Right click on it and select "Modeling Action" -> "Add Externa SAP Hana Service"

10. In the popup dialog box for "Add External SAP Hana Service" set User Provided Service Name as "ServiceParticipant" and provide rest of the information.

11. Open the DB explorer and login to your tenant database. Open the SQL Console and execute following SQL statements

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."VT_CS_STAGEPARTICIPANT" TO MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "TCMP"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

12. Now go back to the Web IDE project and expand the "HDBParticipant" node. Open "syn_participant.hdbsynonym" file by clicking on it. In the database column replace the name of the dabase from "SMF0001LAB" to your database name and save the file.

13. Select the file "syn_participant.hdbsynonym" and just build this selected file. In case everything goes fine it will build the file else try to repeat the privious steps again.

14. Select the flowgraph "VT_to_EXT_Participant.hdbflowgraph" and build the selected file. Similarly select the flowgraph "EXT_to_TCMP_Participant.hdbflowgraph" and build the selected file. If both the files are built successfully the project done. Congrats!

15. Now open "VT_to_EXT_Participant.hdbflowgraph" and in the UI click the "Execute" command. Once the flowgraph is executed the data will be moved from virtual table EXT.VT_CS_STAGEPARTICIPANT to physical table EXT.CS_STAGEPARTICIPANT.

16. Now open "EXT_to_TCMP_Participant.hdbflowgraph" and click on the "Execute" command. After the flowgraph execution is over the data will be moved from EXT.CS_STAGEPARTICIPANT table to TCMP.CS_STAGEPARTICIPANT table.

This completes the migration of the data from remote data source to the TCMP table.

Later we will see how to move the data from TCMP table to SAP Commission main table.
