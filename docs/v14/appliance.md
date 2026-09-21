# Appliance setup

## Appliance basic configuration
1.	Login to **raptor** and check local ip address of **cm**
```bash
cat /etc/hosts
```
[![TZ requests](../images/appl1.webp){ width="90%" }](../images/appl1.webp)
1.	Login to **cm** cli using *ssh* session on **raptor**
```bash
ssh -lcli -p22 cm
```
1.	Run two commands to pause the data aggregation process. In the test environment, enabled aggregation services may cause unintended delays while waiting for several steps to be completed (the second command will require confirmation).
```bash linenums="1"
store run_cleanup_orphans_daily off

store purge_age_period 0
```
[![TZ requests](../images/appl2.webp){ width="80%" }](../images/appl2.webp)
1.	Display network interfaces and routes to verify connectivity.
```bash linenums="1"
show network interface all

show network route default
```
[![TZ requests](../images/appl3.webp){ width="85%" }](../images/appl3.webp)
1.	Notice that the IP address is set to **localhost** or temporary deployment IP, which is specific to the ***IBM Cloud*** environment. Update the IP address to the correct value based on IP address identified in step 1.
```bash
store network interface ip <cm_ip_address>/24
```
[![TZ requests](../images/appl4.webp){ width="85%" }](../images/appl4.webp)
1.	Check DNS resolver configuration. The list will be empty because the appliance is deployed in the cloud.
```bash
show network resolvers
```
1.	In this case, you can retrieve the dynamically assigned list of local servers using the following command.
```bash
show network resolvers_cloud
```
[![TZ requests](../images/appl5.webp){ width="45%" }](../images/appl5.webp)
1.	Test external DNS resolution by pinging a *public hostname*. Do not interrupt the test with **++ctrl+c++**, because this may terminate the ssh connection.
```bash
ping www.ibm.com
```
1.	Display the current system time settings.
```bash
show system clock all
```
[![TZ requests](../images/appl6.webp){ width="45%" }](../images/appl6.webp)
1.	List available time zones. 
```bash
store system clock timezone list
```
1.	Set the system time zone to lab timezone. Wait a few minutes until the change is fully applied. All machines in the lab will use *GMT (Europe/London)* time zone configuration.
```bash
store system clock timezone Europe/London
```
[![TZ requests](../images/appl7.webp){ width="90%" }](../images/appl7.webp)
1.	Re-login to **cli** session on **cm** and confirm new time zone.
```bash
show system clock all
```
[![TZ requests](../images/appl8.webp){ width="55%" }](../images/appl8.webp)
1.	Configure NTP servers (three *pool.ntp.org* servers) and enable the NTP client.
```bash
store system time_server hostname 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org
```
1.	Confirm that they are correctly set.
```bash
show system time_server all
```
[![TZ requests](../images/appl9.webp){ width="55%" }](../images/appl9.webp)
1.	Activate NTP client, execute:
```bash
store system time_server state on
```
1.	Change the central manager host name to **cm**. When asked about cloning, answer *“y”*. This command can take timeout (10 minutes). In that case re-login and confirm than name was set to ***cm.yourcompany.com*** and continue.
```bash
store system hostname cm
```
[![TZ requests](../images/appl10.webp){ width="65%" }](../images/appl10.webp)
1.	Set the domain name
```bash
store system domain demo.guardium
```
[![TZ requests](../images/appl11.webp){ width="70%" }](../images/appl11.webp)
1.	Re-login and confirm that machine prompt is now set to ***cm.demo.guardium.***
[![TZ requests](../images/appl12.webp){ width="60%" }](../images/appl12.webp)
11.	Restart network, insert *Yes* to confirm. It is crucial after machine name change because it intiate the GUI certificate recreation.
```bash
restart network
```
[![TZ requests](../images/appl13.webp){ width="60%" }](../images/appl13.webp)
1.	Because we have multiple cloned appliances in our environment and all share the same Guardium ID (*GID*), assign a **unique** ID to each of them before configuring them to work together, use the following command.
```bash
store product gid 1111
```
1.	All appliances in a centrally managed environment must share the same ***shared secret***. Set it to "*guardium*" on cm.
```bash
store system shared secret guardium
```
1.	Set cm configuration to accept small disk.
```bash
store system small_disk
```
[![TZ requests](../images/appl14.webp){ width="90%" }](../images/appl14.webp)
1.	Increase GUI and CLI session timeouts to avoid unexpected logouts.
```bash linenums="1"
store gui session_timeout 9999

store timeout cli_session 600
```
1.	Display the static host mappings; initially this list is empty.
```bash
support show hosts
```
1.	Add static host entries for all lab machines (**coll1, appnode1, appnode2, kafka1, raptor, sauropod, ceratops**)
```bash linenums="1"
support store hosts <coll1_ip_address> coll1.demo.guardium

support store hosts <appnode1_ip_address> appnode1.demo.guardium

support store hosts <appnode2_ip_address> appnode2.demo.guardium

support store hosts <kafka1_ip_address> kafka1.demo.guardium

support store hosts <raptor_ip_address> raptor.demo.guardium

support store hosts <sauropod_ip_address> sauropod.demo.guardium

support store hosts <ceratops_ip_address> ceratops.demo.guardium
```
[![TZ requests](../images/appl15.webp){ width="90%" }](../images/appl15.webp)
1.	Display the host list again and confirm that all entries are correctly configured.
```bash
support show hosts
```
[![TZ requests](../images/appl16.webp){ width="50%" }](../images/appl16.webp)
1.	Restart the **cm**. If the appliance reports that MySQL is archiving, choose *“No”* to abort the restart and try again later, as shown in the screenshot.
```bash
restart system
```
[![TZ requests](../images/appl17.webp){ width="75%" }](../images/appl17.webp)
28.	Reconnect to the cm and check appliance unit type. The appliance is an *aggregator* and is not yet configured as a *central manager*.
```bash
show unit type
```
[![TZ requests](../images/appl18.webp){ width="40%" }](../images/appl18.webp)
1.	The system has valid license keys loaded, so you can now promote the *aggregator* to be a **Central Manager**. This command may take some time as it activates all services required for managing the appliance group
```bash
store unit type manager
```
[![TZ requests](../images/appl19.webp){ width="55%" }](../images/appl19.webp)
30.	You can proceed with the next part of the lab and return to this *CLI* session once the command above is completed. Then verify that the aggregator has been successfully promoted to a manager.
```bash
show unit type
```
[![TZ requests](../images/appl20.webp){ width="45%" }](../images/appl20.webp)

## Other appliances setup

1.	Sign in to the **coll1** collector from **raptor** and execute a set of configuration commands to prepare the machine for connection to the **cm**.
```bash linenums="1"
store run_cleanup_orphans_daily off

store network interface ip <coll1_ip_address>/24
```
1.	Set the correct time zone (it should match the one configured on the **cm**) and re-login to the **collector**.
```bash
store system clock timezone Europe/London
```
1.	Then configure NTP, as well as the host name and domain. Make sure to confirm that the appliance is cloned when prompted during hostname configuration.
```bash linenums="1"
store system time_server hostname 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org

store system time_server state on

store system hostname coll1

store system domain demo.guardium
```
1.	Re-login and confirm that the command prompt reflects the configured hostname.
[![TZ requests](../images/appl21.webp){ width="75%" }](../images/appl21.webp)
1.	Execute the remaining configuration commands and reboot the appliance.
```bash linenums="1"
restart network

store product gid 2222

store system shared secret guardium

store system small_disk

store gui session_timeout 9999

store timeout cli_session 600

support store hosts <cm_ip_address> cm.demo.guardium

support store hosts <appnode1_ip_address> appnode1.demo.guardium

support store hosts <appnode2_ip_address> appnode2.demo.guardium

support store hosts <kafka1_ip_address> kafka1.demo.guardium

support store hosts <raptor_ip_address> raptor.demo.guardium

support store hosts <sauropod_ip_address> sauropod.demo.guardium

support store hosts <ceratops_ip_address> ceratops.demo.guardium

support show hosts

restart system
```
[![TZ requests](../images/appl22.webp){ width="55%" }](../images/appl22.webp)
1.	Now perform the same steps on the **appnode1** node.
```bash linenums="1"
store run_cleanup_orphans_daily off

store network interface ip <appnode1_ip_address>/24

store system clock timezone Europe/London
```
1.	Re-login and continue on the **appnode1**
```bash linenums="1"
store system time_server hostname 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org

store system time_server state on

store system hostname appnode1

store system domain demo.guardium
```
1.	Re-login to appnode1 and confirm that name is changed
```bash linenums="1"
restart network

store product gid 3333

store system shared secret guardium

store system small_disk

store gui session_timeout 9999

store timeout cli_session 600

support store hosts <cm_ip_address> cm.demo.guardium

support store hosts <coll1_ip_address> coll1.demo.guardium

support store hosts <appnode2_ip_address> appnode2.demo.guardium

support store hosts <kafka1_ip_address> kafka1.demo.guardium

support store hosts <raptor_ip_address> raptor.demo.guardium

support store hosts <sauropod_ip_address> sauropod.demo.guardium

support store hosts <ceratops_ip_address> ceratops.demo.guardium

restart system
```
1.	Then do the same steps on the **appnode2** node.
```bash linenums="1"
store run_cleanup_orphans_daily off

store network interface ip <appnode2_ip_address>/24

store system clock timezone Europe/London
```
1.	Re-login and continue on the **appnode2**
```bash linenums="1"
store system time_server hostname 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org

store system time_server state on

store system hostname appnode2

store system domain demo.guardium
```
1.	Re-login to **appnode2** and confirm that name is changed
```bash linenums="1"
restart network

store product gid 4444

store system shared secret guardium

store system small_disk

store gui session_timeout 9999

store timeout cli_session 600

support store hosts <cm_ip_address> cm.demo.guardium

support store hosts <coll1_ip_address> coll1.demo.guardium

support store hosts <appnode1_ip_address> appnode1.demo.guardium

support store hosts <kafka1_ip_address> kafka1.demo.guardium

support store hosts <raptor_ip_address> raptor.demo.guardium

support store hosts <sauropod_ip_address> sauropod.demo.guardium

support store hosts <ceratops_ip_address> ceratops.demo.guardium

restart system
```
1.	We must do this same for **kafka1** appliance as well
```bash linenums="1"
store run_cleanup_orphans_daily off

store network interface ip <kafka1_ip_address>/24

store system clock timezone Europe/London
```
1.	Re-login and continue on **kafka1**
```bash linenums="1"
store system time_server hostname 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org

store system time_server state on

store system hostname kafka1

store system domain demo.guardium
```
1.	Re-login to **kafka1** and confirm that name is changed
```bash linenums="1"
restart network

store product gid 5555

store system shared secret guardium

store system small_disk

store gui session_timeout 9999

store timeout cli_session 600

support store hosts <cm_ip_address> cm.demo.guardium

support store hosts <coll1_ip_address> coll1.demo.guardium

support store hosts <appnode1_ip_address> appnode1.demo.guardium

support store hosts <appnode2_ip_address> appnode2.demo.guardium

support store hosts <raptor_ip_address> raptor.demo.guardium

support store hosts <sauropod_ip_address> sauropod.demo.guardium

support store hosts <ceratops_ip_address> ceratops.demo.guardium

restart system
```

## Add user and create dashboard

1.	Login to **cm** UI as an **accessmgr** user (use this same password like you used for **cli** access to appliances) and press **Add User** button in **Access manager** view
[![TZ requests](../images/appl23.webp){ width="99%" }](../images/appl23.webp)
1.	Create a new user - **demo**. Set password and description. Expand the **Roles** panel and select: **admin, cli, fam, user** and **vulnerability assess** roles.
[![TZ requests](../images/appl24.webp){ width="99%" }](../images/appl24.webp)
1.	Scroll slightly down, select **Password never expires**, then click **Add** to create the user.
[![TZ requests](../images/appl25.webp){ width="60%" }](../images/appl25.webp)
1.	The newly created user should appear at the bottom of the list. Then, use the **State** toggle for the **guardium** account to deactivate it.
[![TZ requests](../images/appl26.webp){ width="99%" }](../images/appl26.webp)
1.	Sign out of the **accessmgr** account and sign in as the newly created **demo** user.
[![TZ requests](../images/appl27.webp){ width="99%" }](../images/appl27.webp)
1.	In the **Search** field, enter ***License*** to filter the options. Select the **License** view and review the installed license keys. The *Central Manager*, as mentioned earlier, has pre-installed license keys for a centrally managed environment. Functional (*append*) keys enable **DAM, FAM** and **VA** features.
[![TZ requests](../images/appl28.webp){ width="99%" }](../images/appl28.webp)
1.	Navigate to the **Feature Unlock** tab and verify that the additional features have been loaded into the system. Note that this does not mean they have been activated.
[![TZ requests](../images/appl29.webp){ width="99%" }](../images/appl29.webp)
1.	Next, select **Definition Export/Import**, go to the **Import** tab, choose the report definition file `exp_report_two_of_training_dasboard.sql` from your exports folder (student materials), and click **Upload** file.
[![TZ requests](../images/appl30.webp){ width="99%" }](../images/appl30.webp)
1.	After a moment, the *Full SQL (Training)* report should appear in the **Uploaded files** section. Select it and click **Import**.
[![TZ requests](../images/appl31.webp){ width="99%" }](../images/appl31.webp)
1.	After a moment, a confirmation message should appear indicating the report was successfully imported.
[![TZ requests](../images/appl32.webp){ width="99%" }](../images/appl32.webp)
1.	Repeat the import process, but this time use the `exp_default_policy.sql` file. This will load the policy named *Default bootcamp policy*.
[![TZ requests](../images/appl33.webp){ width="99%" }](../images/appl33.webp)
1.	To create a dashboard, select the **My Dashboards** icon from the vertical menu, then choose **Create New Dashboard**. Edit the name (pencil icon) to *Training* and click **Save**. Next, click **Add Report** to open the report selection window and add three reports: the newly imported *Full SQL (Training)* and the built-in *SQL Errors* and *S-TAP and External S-TAP Statistics*. Reports are added to the dashboard upon selection from the list.
[![TZ requests](../images/appl34.webp){ width="99%" }](../images/appl34.webp)
!!! warning "Uwaga"
    Use **demo** account for all UI activities in the bootcamp labs unless a different one is explicitly required.

## Register appliances

1.	From **cm** UI open **Central Management** application and select **Register unit** button from **Actions** list. Notice that there are no appliances managed by our *Central Manager* yet. Register unit pop-up will provide the possibility to insert **coll1** Unit IP (select appropriate one) and default communication port (8443) then press **Register** button.
[![TZ requests](../images/appl35.webp){ width="99%" }](../images/appl35.webp)
1.	After a longer wait (be patient), the view should refresh and the **coll1** collector will appear in the managed appliances list. Its presence on the list does not yet mean full synchronization—this process may take a few more minutes.
[![TZ requests](../images/appl36.webp){ width="99%" }](../images/appl36.webp)
1.	Sign in to the **coll1** via CLI and notice that the appliance mode has changed from Standalone to Managed.
```bash
show unit type
```
[![TZ requests](../images/appl37.webp){ width="45%" }](../images/appl37.webp)
1.	Repeat the full appliance registration process on **appnode1**, **appnode2** and **kafka1** and confirm that all appliances are visible from the **cm** *Central management* view.
[![TZ requests](../images/appl38.webp){ width="99%" }](../images/appl38.webp)

## Central Manager patching

!!! info "Screenshots"
	Screenshots in this section may not exactly match the patch numbers available in your environment. The labs are frequently updated, so there may be some inconsistencies between this documentation and what is present on the machines.
1.	In **cm** UI, open **Installed Patches** to confirm which patches are currently installed.
[![TZ requests](../images/appl39.webp){ width="99%" }](../images/appl39.webp)
1.	On **raptor**, there are several additional patches available that we will install on the **cm**. Patches are files with the `.sig` extension.
```bash
ls -l /opt/guardium_tz_bootcamp_automation/upload/source_files/appliances/patches/
```
[![TZ requests](../images/appl40.webp){ width="99%" }](../images/appl40.webp)
1.	The directory also contains a `patch_order.txt` file that specifies the correct order for applying patches. In the next step, you will provide a list of patches for installation - make sure to follow the installation sequence defined in this file.
```bash
cat /opt/guardium_tz_bootcamp_automation/upload/source_files/appliances/patches/patch_order.txt
```
[![TZ requests](../images/appl41.webp){ width="99%" }](../images/appl41.webp)
1. Login to **cm** cli and upload all patches to it using flow below. Execute command:

    ```bash
    store system patch install scp
    ```

    !!! abstract "Insert:"
        - raptor IP address: **raptor.demo.guardium**
        - user name: **root**
        - patch path: `/opt/guardium_tz_bootcamp_automation/upload/source_files/appliances/patches/*.sig`
        - Port: **2223**
        - password for root account on raptor

    Finally, you will be asked for order of patch installation, it must be comma separated list based on patch numbering on the displayed list (for example 4,3,2,1), please use correct order provided in patch_order.txt file mentioned above. Some other patches will be on the list - ignore them, they are related to previous patch processes.

    !!! note
        You will be asked for patch **9997** reinstallation confirmation - *accept this!*

    [![TZ requests](../images/appl42.webp){ width="99%" }](../images/appl42.webp)

1. Monitor installation progress using the relevant patch status commands. Some patches temporarily stop backend services, so one view may show errors. Check the list of downloaded patches on **cm**
```bash
show system patch available
```
[![TZ requests](../images/appl43.webp){ width="99%" }](../images/appl43.webp)
1.	All freshly uploaded patches will be scheduled, and you can monitor the installation process using commands:
    ```bash
    show system patch installed
    ```
    [![TZ requests](../images/appl44.webp){ width="99%" }](../images/appl44.webp)
    and
    ```bash
    show system patch status
    ```
    [![TZ requests](../images/appl45.webp){ width="99%" }](../images/appl45.webp)
    The first command can produce errors if some patch installation tasks require stop collector backend (MySQL). In this case you can use the second one to check the progress.

    !!! note
        Process can take several minutes (depending on patches applied during this course instance) and you must wait that all scheduled patches will be installed successfully. Some patches can force collector restart and you must re-login.

1.	Confirm that all patches are installed. You can noticed warning at patch **9997** – you can ignore it.
```bash
show system patch installed
```
[![TZ requests](../images/appl46.webp){ width="99%" }](../images/appl46.webp)
1.	It is good practice to schedule the re-installation of patch **9997** at the beginning of every appliance patching process to ensure that the latest version available on the **cm** is installed.

## Collectors patching

1.	Patching the remaining appliances can be managed centrally from the **cm**. In the **cm** UI, open **Patch Management**.
    From the *Available patches* section, select patch **9997** (first in the `patch_order.txt` list), then select your collectors in the *Guardium systems* section. Click **Install**, which opens a popup where you can schedule the patch installation.
    Choose *NOW* for immediate deployment and confirm by clicking **Install**. A confirmation popup should appear indicating that the installation job has been successfully created.
    [![TZ requests](../images/appl47.webp){ width="99%" }](../images/appl47.webp)
1.	Repeat these steps for all other patches mentioned in `patch_order.txt` list.
1.	To check the patch installation status, select one of the appliances and choose *Show all installed patches*. This will open a window displaying the list of patches on that system. You should see the newly scheduled patches listed with various installation statuses. Confirm that all patches have been installed.
    [![TZ requests](../images/appl48.webp){ width="99%" }](../images/appl48.webp)

## Collector default policy

1.	A newly deployed collector has a default policy assigned that ignores all activity. To start collecting traffic, you need to install a meaningful policy—use the one previously imported.
    In the **cm** UI, open **Security Policies**, filter the view to show only policies (uncheck *Include templates*), select **Default bootcamp policy**, and from the actions list choose **Install**.
    In the popup, select your **coll1.demo.guardium** and choose the installation type *Install and override*, then confirm with **OK**. You should receive confirmation that the policy has been installed on the **coll1**.
    Do not install this policy on other appliances, as they will be used for a different purposes.
    [![TZ requests](../images/appl49.webp){ width="99%" }](../images/appl49.webp)

1.	Open **Central management** view on **cm**. Verify that we have our policy installed on collector **coll1** and generic one is still applied on the rest appliances.
[![TZ requests](../images/appl50.webp){ width="99%" }](../images/appl50.webp)

## Backup, configuration profiles (optional)

1.	To configure scheduled appliance backup open in cm GUI the System Backup view

    !!! abstract "Use this information to fill in a form:"
        *Endpoint URL*: **s3.ams03.cloud-object-storage.appdomain.cloud**

        *Access Key ID*: **2a20bcab1bfc4503bc5466cd549aba6e**

        *Secret access key*:  **962fc95e0b20d77258e86279561011804c1f3dd0e6348ad9**

    [![TZ requests](../images/appl51.webp){ width="99%" }](../images/appl51.webp)

1.	Use **Test connection** button to check correctness of inserted values and then close pop-up message and **Save** configuration. Do not run the backup on the **cm**; we will do it in a moment on the **coll1**.
[![TZ requests](../images/appl52.webp){ width="99%" }](../images/appl52.webp)

1. In the centrally managed environment we can spread some configuration settings across managed appliances to avoid repetion of this same tasks in the large installation. Let’s do this with backup configuration. Open **Distribute Configuration Profiles** view and add a new one.
[![TZ requests](../images/appl53.webp){ width="99%" }](../images/appl53.webp)

1.	Insert the **Name**, for example – *Backup configuration* and press **Next** button
[![TZ requests](../images/appl54.webp){ width="99%" }](../images/appl54.webp)

1.	In the *What to distribute* section create a new one and select **System Backup**
[![TZ requests](../images/appl55.webp){ width="99%" }](../images/appl55.webp)

1.	Similar to backup configuration for **cm**, define this same bucket and **Save** configuration

    !!! abstract    
        *Endpoint URL*: **s3.ams03.cloud-object-storage.appdomain.cloud**
        *Access Key ID*: **2a20bcab1bfc4503bc5466cd549aba6e**
        *Secret access key*:  **962fc95e0b20d77258e86279561011804c1f3dd0e6348ad9**

    Then press **Bucket Name** button and select *bucket-2znplpxrvg22c3v* bucket. Select only *Configuration* checkmark.
    [![TZ requests](../images/appl56.webp){ width="99%" }](../images/appl56.webp)

1.	Just created configuration asset will appear on the list. Press **Next** button
[![TZ requests](../images/appl57.webp){ width="99%" }](../images/appl57.webp)
1.	In the *Where to distribute* panel select *All Collectors* group and move it to *Selected groups* section. Then press **Next** button.
[![TZ requests](../images/appl58.webp){ width="99%" }](../images/appl58.webp)
1.	**Save** configuration and press **Run Now** button to send configuration to the collector
[![TZ requests](../images/appl59.webp){ width="99%" }](../images/appl59.webp)
1.	Monitor the progress of profile distribution. Finally the *Review distribution results* section should appear and confirm that **coll1** and the rest of collectors are successfully synchronized.
[![TZ requests](../images/appl60.webp){ width="99%" }](../images/appl60.webp)
1.	Login to **coll1** UI and confirm that **System Backup** configuration has been successfully set. Press **Run Once Now** button
[![TZ requests](../images/appl61.webp){ width="99%" }](../images/appl61.webp)
1.	You can check status of task in **Aggregation/Archive Log** report (on **coll1**)
[![TZ requests](../images/appl62.webp){ width="99%" }](../images/appl62.webp)
1.	Ask instructor to display content of bucket
[![TZ requests](../images/appl63.webp){ width="99%" }](../images/appl63.webp)

## guardcli accounts (optional)

1.	**cli** accounts are shared ones. To manage cli access accountable the **Guardium** introduce on each appliance the nine additional accounts from **guardcli1** to **guardcli9** (they have *OTP* set to **guardium**). Let’s us configure accountable access to **cm cli**.
1.	From **raptor** connect to **cm** using **guardcli1** shared account (instead **cli** one) and reset password from OTP “guardium” to desired one if you will asked for change (must be strong, you can notice a bug and could be forced to change password two times), so execute from **raptor**:
```bash
ssh -l guardcli1 -p22 cm
```
[![TZ requests](../images/appl64.webp){ width="79%" }](../images/appl64.webp)
1.	Try to execute any administration command, for example:

    ```bash
    show network int all
    ```

    [![TZ requests](../images/appl65.webp){ width="99%" }](../images/appl65.webp)

1.	You will be informed that you cannot execute commands before reauthentication using the named UI account. Then activate a full access to the CLI using command below (provide correct password for **demo** UI account). Then execute any administration command to confirm this ability now.
```bash linenums="1"
set guiuser demo

show network int all
```
[![TZ requests](../images/appl66.webp){ width="79%" }](../images/appl66.webp)
1.	Add to your *Training* dashboard on **cm** the **Detailed Guardium User Activity Trail** report. If **Add Report** button is inactive you must activate it by clicking **Edit mode**

    [![TZ requests](../images/appl67.webp){ width="99%" }](../images/appl67.webp)

1.	Open just added report in the **Dashboard** on **cm**. Notice that your **guardcli1** session is marked as belonging to **demo** UI account and we can track activity correctly.
[![TZ requests](../images/appl68.webp){ width="99%" }](../images/appl68.webp)
1.	On the fresh installed appliance all **guardcliX** accounts have the well known OTP password – **guardium**. The good administration approach is access control to all proxy accounts. A good practice is to block those accounts or change the password to prevent unauthorized login attempts. To list **guardcliX** account status execute in CM cli session:
```bash
show guarduser_state all
```
1.	Disable all proxy accounts - except just used above the **guardcli1** - by execution commands:

    ```bash
    store guarduser_state disable <guardcli[2-9]>
    ```

    Proxy account should be now set this way

    [![TZ requests](../images/appl69.webp){ width="69%" }](../images/appl69.webp)

## Appendix
!!! note "Dependencies:"

    **IBM Cloud** URL for COS. It is COS bucket belonging to **zibi**. In case of self-managed training or led by another instructor the backup lab should refer to appropriate, accessible COS bucket.

    <https://cloud.ibm.com/objectstorage/crn%3Av1%3Abluemix%3Apublic%3Acloud-object-storage%3Aglobal%3Aa%2F8d537ccde33a156ac84a2890733b572d%3A7748e344-6c18-405c-be45-50b4742ee67c%3A%3A>


!!! note "Resources:"

!!! note "Instructor notes:"
    



