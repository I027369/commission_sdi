# commission_sdi
Web IDE project for data integration to SAP Commission using SDI

This repository explain how to import SDI project in your local workspace and create a working data integration solution using SDI. Following are the steps:-

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

8. Log in the Web IDE. Import the 
