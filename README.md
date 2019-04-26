# commission_sdi
Web IDE Project for Data Integration with SAP Commissions Using SDI:

This article explains how to import the SDI project in your local workspace and create a working data integration solution using SDI. Once all the steps are completed successfully, you will be able to build a Web IDE project which will move the data from the remote data source (in this example: Flat File) to a physical table in EXT Schema. This is followed by moving the data from EXT schema to the TCMP the Schema. In this example, we will use the CS_STAGEPARTICIPANT file for data integration. We will use Extract, Load, and Transform pattern.

Following are the steps:

1. Connect the DP Agent to the SDI server and enable the File Adapter.
Refer to these videos if you are performing this step for the first time:

    https://www.youtube.com/watch?v=4anKSVRrHh0&t=379s

    https://www.youtube.com/watch?v=8j6lD7lP5B0

    Similarly, in the Linux setup, check the following in the DP Agent File Adapter Configuration:

    a.	Root directory

    b.	Format root directory

    c.	Enter access token

2. Download the CS_STAGEPARTICIPANT.txt in the root directory (as set up in the previous step). This file contains sample data.  It acts as the source data in this example.

3. We also need to create the metadata file(.cfg) for the source data file to migrate the data. Execute the following command to create the metadata file and replace the root directory information in the previous step:
"createfileformat.bat -file <<rootdirectory>>\CS_STAGEPARTICIPANT.txt -cfgdir <<rootdirectory>> -firstRowAsColumnName true"

  You can find the "createfileformat" executable in the "AgentUtil" folder where the DP Agent is installed for both Windows and Linux environment. In Linux, use "createfileformat.sh".

  The resulting configuration file will look something like CS_STAGEPARTICIPANT.txt.cfg. (This file is attached only for reference. Do not use this file as is because it will not work without modifications in your environment.)

 4. Create a remote source in the HANA DB as explained in the video.
    Note: While creating the remote source in the DB Explorer, you need to provide a SharePoint login in HANA. You can use any dummy username and password (though it is not relevant, you cannot save the configuration without providing a dummy value).

5. Create a virtual table named VT_CS_STAGEPARTICIPANT in EXT Schema as shown in the video. Check if you can view the data using the Open Data feature of the DB Explorer.

6. Create a physical table in the EXT schema named CS_STAGEPARTICIPANT. It should have the same structure as the VT_CS_STAGEPARTICIPANT virtual table. You can use a similar command such as, CREATE TABLE  "EXT"."CS_STAGEPARTICIPANT" ,  "EXT"."VT_CS_STAGEPARTICIPANT". The idea is to create a flowgraph and first move the data from the virtual table (created in previous step) to the physical table created in this step.

7. Download the MigrateParticipant.zip file in any folder. This is the Web IDE project which we will import in next step.

8. Log in the Web IDE. Select the "Workspace" in the Development area and right-click to "Import File or Project". Click Browse to navigate and locate MigrateParticipant.zip and select it. Retain the default settings and click OK.This step creates a project in the Web IDE workspace.

9. Right click on the project and go to "Project Settings". Select the Space entry and Save the setting. (There should be only one Space listed in the drop-down.) When we deploy the project, this space information is used to create Run Time Objects.

10. Expand the project. Select the "HDBParticipant" node and right-click to select "Modeling Action" > "Add Externa SAP Hana Service". We need to create this service to access the table in EXT and TCMP Schema. Web IDE project creates a HDI container to access the HANA DB. It creates technical users to access the database(HDI container). These users only have access to the HDI container. We need to: 1. Create the service to access data from EXT and TCMP schema from Web IDE artifacts (which this step will accomplish). 2. Create synonym for the EXT and TCMP tables as Web IDE artifacts (flowgraphs) can access these tables only via synonym and not directly. We will handle synonym in later steps.

10. In the popup dialog box for "Add External SAP Hana Service", set User Provided Service Name as "ServiceParticipant" and provide rest of the information.

11. Open the DB Explorer and Login to your tenant database. Open the SQL Console and execute the following SQL statements:

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."VT_CS_STAGEPARTICIPANT" TO MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "EXT"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

GRANT SELECT, INSERT, UPDATE, DELETE  ON "TCMP"."CS_STAGEPARTICIPANT" to MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO;

As mentioned earlier, the Web IDE project creates technical users. In this project, it creates the "MIGRATEPARTICIPANT_HDI_HDBPARTICIPANT_1#OO" user, and therefore in this step, access is provided to this user to access EXT and TCMP tables. Note: The Web IDE flowgraphs can only access these table via synonyms which we will create in next step.

12. Go back to the Web IDE project and expand the "HDBParticipant" node. Open the "syn_participant.hdbsynonym" file by clicking on it. In the database column, replace the name of the dabase from "SMF0001LAB" to the required database name (your database), and save the file.

13. Select the file "syn_participant.hdbsynonym" and build the selected file. If the build is successful, it will create three synonyms to be used in this project. If the build is not successful, try to repeat the previous steps again.

14. Select the flowgraph "VT_to_EXT_Participant.hdbflowgraph" and build the selected file. Once executed, this flowgraph will move the data from the virtual table to the physical table in EXT schema i.e. from "VT_CS_STAGEPARTICIPANT to CS_STAGEPARTICIPANT.
Similarly, select the flowgraph "EXT_to_TCMP_Participant.hdbflowgraph" and build the selected file. Once executed, this flowgraph will move the data from the tables in EXT schema to TCMP schema tables.
If both the files are built successfully, the project is built! Congrats!

15. Open "VT_to_EXT_Participant.hdbflowgraph", and click "Execute" command on the UI. Once the flowgraph is executed, the data will be moved from the virtual table EXT.VT_CS_STAGEPARTICIPANT to the physical table EXT.CS_STAGEPARTICIPANT.

16. Now open "EXT_to_TCMP_Participant.hdbflowgraph" and click on the "Execute" command. After the flowgraph is executed, data will be moved from the EXT.CS_STAGEPARTICIPANT table to the TCMP.CS_STAGEPARTICIPANT table.

This completes the migration of the data from a remote data source to the TCMP table.

Next, we will see how to move the data from the TCMP table to the SAP Commissions main table. For this, we will create a workflow using the OData API from the SAP Commissions solution.
