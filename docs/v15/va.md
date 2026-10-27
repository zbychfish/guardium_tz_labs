# Vulnerability Assessement configuration

## VA Feature Flag

1.	Starting with version **12.2**, some **Guardium** features must be enabled through a mechanism called *feature flags*. Activating a *feature flag* is a two-step process. First, the appropriate feature options file must be loaded, which has already been completed in our environment.
![TZ requests](../images/va1.webp){ width="99%" }
 
1.	To activate the *feature flag*, run the following command from the **cm** cli. The *VULNERABILITY_MANAGEMENT* flag enables an additional UI view that will be used in the later sections of this lab.
```bash
grdapi enable_disable_feature_flag flagName=VULNERABILITY_MANAGEMENT action=enable
```
1.	And confirm operation
```bash
grdapi list_feature_flags
```
![TZ requests](../images/va2.webp){ width="99%" }

## Update VA tests

1.	From browser go to *Fix Central* (<https://ibm.com/support/fixcentral/>) and look for *DPS* updates for **Guardium 12.2**
![TZ requests](../images/va3.webp){ width="99%" }
 
1.	In *DPS* section locate the latest quarter update (position 4 in my case, Q4’25 update) and the latest *Rapid Response* update (if exists) related to quarter one (position 1 my case). Download this/these file/files.
![TZ requests](../images/va4.webp){ width="99%" }

1.	Unpack zip file/files. Inside each archive you should have file with `enc` extension.
![TZ requests](../images/va5.webp){ width="99%" }

1.	Open **Customer Uploads** view on **cm** and press **Browse** button. Select quartely *DSP* update file and then use **Upload** button to load it to the engine.
![TZ requests](../images/va6.webp){ width="99%" }

1.	Import *DPS* section will be refreshed and the uploaded file should appear. Press the ![TZ requests](../images/va7.webp){ width="20" } icon to start *DPS* import – popup with message “*Import DPS is in progress …*” should appear (wait a while to get pop-up). Close it and wait for next pop-up. It can takes even few minutes.
![TZ requests](../images/va8.webp){ width="99%" }
1.	Accept the second pop-up with information that *DPS* import started and notice that message below of *DPS Upload* section is changed and refers to new *DPS* file and informs that process is running. If message will not appear, refresh browser tab and check it again.
![TZ requests](../images/va9.webp){ width="99%" }

1.	You can go back to **Customer Upload** view after few minutes and notice that *DPS import* process finished successfully. You can continue lab; there is no demand to have the latest updates. Sometimes *DPS import* takes hours.
![TZ requests](../images/va10.webp){ width="69%" }
 
1.	If you have Rapid Response update repeat the upload procedure for it according to steps from **4** to **7**. You process the rapid update when quarter one is correctly imported

## VA roles scripts

1.	Connect to **cm** cli and execute command to start a web based **fileserver** with access to files stored on appliance filesystem. The command parameters are: an *IP address* which points that connection will be possible only from our windows machine and the second one sets that service will be automatically shutdown *after 1 hour*. Do not press ++enter++ when **fileserver** will be started.
```bash
fileserver <ceratops_ip> 3600
```
![TZ requests](../images/va11.webp){ width="99%" }
 
1.	In **RDP** session on **ceratops** start browser and go to URL – <https://cm.demo.guardium:8445>
![TZ requests](../images/va50.webp){ width="79%" }
 
1.	Select *Access logs* and move to `log/debug-logs/gdmonitor_scripts` directory. This folder contains the scripts to create access role for all supported by VA databases. Find `gdmmonitor-postgres.sql` and click it to download the linked file.
![TZ requests](../images/va51.webp){ width="99%" }
 
1.	In the **Windows Explorer** go to *Downloads* directry and open just downloaded file using **Notepad**.
![TZ requests](../images/va52.webp){ width="99%" }

1. For *PostgreSQL*, the script contains a simple definition of the **gdmmonitor** group with a privilege that allows it to retrieve configuration settings. For other database platforms, the script may be more complex. However, the goal remains the same: provide sufficient permissions to perform vulnerability assessments while following the least-privilege principle and avoiding excessive privileges.
![TZ requests](../images/va53.webp){ width="99%" }

1.	Back to **cm** cli session and stop **fileserver** – press ++enter++ in the ssh session.
	
## Configure VA technical account

1.	On **raptor** connect to *PostreSQL* as superuser and create a new user account (**sqlguard**) and a new group (**gdmmonitor**) (set your own password!). Assign to the group limited rights to read database configuration only (based on script reviewed above).
```sql linenums="1"
su - postgres

psql

CREATE USER sqlguard WITH ENCRYPTED PASSWORD '<your_password>';

CREATE GROUP gdmmonitor;

ALTER GROUP gdmmonitor ADD USER sqlguard;

GRANT pg_read_all_settings TO gdmmonitor;

GRANT SELECT ON pg_authid TO gdmmonitor;

\q

exit
```

## Configure VA for PostgreSQL

1.	Open Datasource Definitions and click :material-plus-circle: to add a new datasource.
![TZ requests](../images/va55.webp){ width="99%" } 

1. Create a new datasource for VA named **postgres on raptor** using the parameters below. Then click **Save** to save the datasource and enable **Test Connection** button. Test the connection and verify that the following message appears:  *:material-checkbox-marked-circle: Connection successful*. Finally, **Close** the window.

    !!! note "datasource definition:"
        - Application type: **Security Assessment**
        - Name: **postgres on raptor**
        - Database type: **PostgreSQL**
        - Share datasource: :material-checkbox-marked:
        - Credential type: **Assign credentials**
        - User name: **sqlguard**
        - Password: **&lt;define in the previous task>**
        - Host name/IP: **raptor.demo.guardium**
        - Port number: **5432**
        - Database: **postgres**

    ![TZ requests](../images/va56.webp){ width="79%" } 

1. The newly created datasource should now appear in the Datasource Definitions list.
![TZ requests](../images/va57.webp){ width="99%" } 

2. Open **Assessment Builder** (Security sssessments) view on **cm** and **Create assessment** definition.
![TZ requests](../images/va54.webp){ width="99%" } 

1.	Insert values listed below and press **Next** button.

    !!! note ""
        - Name: **postgres on raptor**
        - Datasource type: **POSTGRESQL**
        - Automatically add all future CVE or APAR tests after DPS uploaded: :material-checkbox-marked:
![TZ requests](../images/va58.webp){ width="99%" }
 
1. In the **Add Datasources** section, click **Add datasource**. Select the newly created **postgres on raptor** datasource definition, then click **Add**.
![TZ requests](../images/va59.webp){ width="99%" }

1.	The selected datasource should now appear in the **Create assessment** view. Click **Next** to continue.
![TZ requests](../images/va60.webp){ width="99%" }
 
1.	To select the tests to include in the assessment, click **Add tests**.
![TZ requests](../images/va61.webp){ width="99%" }
 
1.	The new view displays all tests available for the database type associated with the selected datasource. In this case, the available *PostgreSQL* tests are displayed. To select all available tests, select the checkbox next to *Test ID* <span class="mb">1</span>, then click **Select all** <span class="mb">2</span>.
![TZ requests](../images/va62.webp){ width="99%" }
 
1.  All available tests are now selected. Click **Add** to confirm your selection.
![TZ requests](../images/va63.webp){ width="99%" }

1.	The **Create assessment** view should now display the selected tests. Click **Create** to save the new VA configuration.
![TZ requests](../images/va64.webp){ width="99%" }
 
1.	To execute just created *assessment*, press run :material-play-circle: icon. The pop-up will inform that we just started *assessment instance* and its execution can be monitored in *Guardium Job Queue* report.
![TZ requests](../images/va65.webp){ width="99%" }
 
1.  The status of the running assessment can be monitored directly in the current view. Click :material-refresh: to confirm that the assessment has completed.
![TZ requests](../images/va66.webp){ width="99%" }

1.	To review the results, use the actions menu :material-dots-vertical: icon and select **View scan results**.
![TZ requests](../images/va67.webp){ width="99%" }

1.	Review *VA assessment* report and close it.
![TZ requests](../images/va27.webp){ width="99%" }

## Findings remediation

1.	In **cm** UI open again the result set from the last *PostgreSQL* assessment. Press **View Result** when vulnerability assessment - **postgres on raport** is selected.
![TZ requests](../images/va67.webp){ width="99%" }

1.	Look for test – *Ensure the pgcrypto extension is installed*. This extension allows implement column based data encryption. Unfortunately, it is not enabled in our **raptor** *PostgreSQL instance*.
![TZ requests](../images/va28.webp){ width="99%" } 

1.	To enable the functionality mentioned in the test, log in on **raptor** to the database and execute the **CREATE EXTENSION** statement.
```sql
su - postgres

psql

CREATE EXTENSION IF NOT EXISTS pgcrypto;

/q

exit
```

1.	Run again our **postgres on raptor** *VA assessment* and wait for execution completeness.
 ![TZ requests](../images/va65.webp){ width="99%" }
 

1.	Open assessment results and notice that Test passing result has been changed. To find a difference you can press **Compare with other results** link and select previous execution with lower score. Press **Go** button.
![TZ requests](../images/va31.webp){ width="99%" }

1.	Displayed pop-up report identifies that now *pgcrypto* library test has been passed.
![TZ requests](../images/va32.webp){ width="99%" }
	
## Setup remote VA scanner (optional)

1.	Open the **Security assessments** list again, from the actions menu :material-dots-vertical: and select **Run on scanner**.
![TZ requests](../images/va68.webp){ width="99%" }

1. After a while, the status of the assessment **postgres on raptor** will change *Status* to **In-progress**. This is expected behavior, as the *VA Scanner* has not been deployed yet.
![TZ requests](../images/va69.webp){ width="99%" }

1.	Open **VA Scanner Job Queue** view and notice that a new task is scheduled and it is **WAITING** for execution. In this case the task will be served by remote scanner which we must deploy first.
![TZ requests](../images/va34.webp){ width="99%" }
 
1.	To deploy scanner we need two pieces of information. First one is *VA appliance UI certificate*. In our case we are using **cm** for *VA*, so let us get it public certificate. On **sauropod** execute these commands:
```bash linenums="1"
mkdir -p /opt/vascanner/certs

openssl s_client -connect cm.demo.guardium:8443 -showcerts </dev/null 2>/dev/null|openssl x509 -outform PEM > /opt/vascanner/certs/vascanner.pem
```

1.	The `vascanner.pem` file should look like shown below:
```bash 
cat /opt/vascanner/certs/vascanner.pem 
```
![TZ requests](../images/va37_v15.webp){ width="79%" }
 
1.	Login to **cm** cli and create a new *API key* for *vascanner*.
```bash
grdapi create_api_key name=vascanner
``` 
![TZ requests](../images/va38.webp){ width="99%" }
Write down the encoded API key, we need it soon.

1.	On **sauropod** login to the *IBM container registry* and check. The entitlement key is available in the `container_registry.txt` file in training materials.
```bash
podman login cp.icr.io -u cp -p <ibm_registry_key>
```
![TZ requests](../images/va39.webp){ width="99%" }
 
1.	Pull *VA scanner* image on **sauropod**.
```bash
podman pull cp.icr.io/cp/ibm-guardium-data-security-center/guardium/vascanner-12.2.0/va-scanner:vascanner-v12.2.0
```
![TZ requests](../images/va40.webp){ width="99%" }

1.	Check that image is correctly pulled and wrote down information about *image id*.
```bash
podman images
```
![TZ requests](../images/va41.webp){ width="99%" }
 
1.	Create config file in the `/opt/vascanner/config` on **sauropod**.

    !!! note "Inside 4 variables must be set like that:"
        - GDP_HOST=**cm.demo.guardium**
        - GDP_HOST_PORT=**8443**
        - CLIENT_API_KEY=&lt;encoded API Key from point 8>
        - VA_AGENT_NAME=**VA_SCANNER_ON_SAUROPOD**

    ![TZ requests](../images/va42.webp){ width="89%" }
 
1.	Now we are ready to start a *scanner* (insert correct *image id*)
```bash
podman run --network host -d --replace --env-file /opt/vascanner/config --name va-scanner-sauropod -v /opt/vascanner/certs:/var/vascanner/certs <image ID from point 9>
```

1.	You can monitor the container status using:
```bash
podman ps -a
```
![TZ requests](../images/va43.webp){ width="99%" }
After one minute, even less, the container will stop working (it is expected).
 
1.	Check again the **VA Scanner Job Queue** view in **cm** UI and notice that job is completed and result set is consumed by *VA engine*.
![TZ requests](../images/va44.webp){ width="99%" }

1.  The status in **Security assessments** should also confirm that the requested assessment completed successfully.
![TZ requests](../images/va70.webp){ width="99%" }

1.	You can also check container logs on sauropod/
```bash
podman logs va-scanner-sauropod
```
![TZ requests](../images/va45.webp){ width="99%" }

## VA findings dashboard

1.	If the **VULNERABILITY_MANAGEMENT** *feature flag* is enabled on the **cm**, a new **Vulnerability Management** view becomes available. This view provides a quick overview of the security posture of the monitored database engines and centralizes vulnerability assessment findings. The charts provide a visual overview of the identified findings, categorized by critical, high, and other severity levels, as well as grouped by database engine type. Below the charts, you can review the findings in greater detail and examine the specific issues detected by the assessment.
![TZ requests](../images/va46.webp){ width="99%" }
 
1.	Selecting a test result from the list opens detailed information about the issue, including remediation guidance, and also allows you to add comments.
![TZ requests](../images/va47.webp){ width="99%" }

1.  To see remediated issue with missing encryption plugin **Clear filters** applied in the view.
![TZ requests](../images/va71.webp){ width="99%" }

1.  You can now see all available tests and apply a filter using the :material-filter: icon. Under *Status*, select **Fixed** and **Apply**.
![TZ requests](../images/va72.webp){ width="99%" }

1.  To see remediated issue with missing encryption plugin **Clear filters** applied in the view.
![TZ requests](../images/va73.webp){ width="99%" }

<!-- 
BUG IN 12.2.4
1.	The new view can also be used to manage the remediation process by updating the finding status and assigning remediation tasks or other actions to users with Guardium accounts. This lightweight workflow can be a good fit for smaller organizations or serve as a temporary solution before implementing a full-scale remediation process based on exceptions and formal workflow management.
![TZ requests](../images/va48.webp){ width="99%" }

1.	Findings can also be reviewed from the perspective of configured data sources (assets) or by examining the results of individual vulnerability assessment processes that have been configured and executed.
![TZ requests](../images/va49.webp){ width="99%" } -->

## Appendix

!!! note "Dependencies:"
    Deliverables from this lab are referred in other, it should be fully followed.

!!! note "Resources:"
    - Registry entitlement (use own or ask instruktor to provide one) <https://myibm.ibm.com/products-services/containerlibrary>
    
!!! note "Instructor notes:"

 


