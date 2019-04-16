# commission_sdi
Web IDE project for data integration to SAP Commission using SDI

This repository explain how to import SDI project in your local workspace and create a working data integration solution using SDI. Following are the steps:-

1. Connenct the DP agent to the SDI server and enable the File Adapter
2. Create a remote source in the Hana DB. 
  Follow this video in case you are doing it for the first time. It explains how to create this remote source for Windows source
  
    https://www.youtube.com/watch?v=4anKSVRrHh0&t=379s
    https://www.youtube.com/watch?v=8j6lD7lP5B0
    
    Similarly in the linux system setup the 
    a.	Set the root directory
    b.	Set the format root directory
    c.	Enter access token

    Please note - for SharePoint login use any dummy user and password (though it is not relevant, you can save the configuration without putting dummy value)
    
3. Download the CS_STAGEPARTICIPANT.txt and CS_STAGEPARTICIPANT.txt.cfg in the root directory in the source system which you setup in the previous step. Please note in the linux environment change the owner of the files to the DP Agent user.
4. 
