-------------------------------
Right-sizing VMs with Prism Ops
-------------------------------

.. figure:: images/operationstriangle.png

Prism Ops brings smart automation to our customer’s daily IT operations. The typical operations workflow is a continuous cycle of monitoring, analyzing and taking action where necessary. Prism Ops mirrors traditional IT Admin's workflows to improve operations efficiency. With Prism Ops, IT Admins are able to connect insights from machine data to automate this typical flow using the power of the machine learning engine X-FIT and the X-Play automation engine.

In this lab you will learn how Prism Ops can help IT Admins monitor, analyze and automatically act when a VM's memory resource is constrained.

Lab Setup
+++++++++

#. Open your **Prism Central** and navigate to the **VMs** page. Note down the IP Address of the **GTSPrismOpsLabUtilityServer**. You will need to access this IP Address throughout this lab.

   .. figure:: images/init1.png

#. Open a new tab in the browser, and navigate to http://`<GTSPrismOpsLabUtilityServer_IP_ADDRESS>`/alerts [example http://10.42.113.52/alerts]. It is possible you may need to log into the VM if you are the first one to use it. In that case, fill out the **Prism Central IP** (from the assignment spreadsheet, not the IP we just documented), **Username** and **Password** and click **Login**.

   .. figure:: images/init2.png

#. Once you have landed on the alerts page, leave this tab open. It will be used in a later portion of this lab.

   .. figure:: images/init2b.png

#. In a new tab, navigate to http://`<GTSPrismOpsLabUtilityServer_IP_ADDRESS>`/ to complete the lab from [example http://10.42.113.52/]. Use this tab to complete the lab, unless specifically directed otherwise.

   .. figure:: images/init3.png

Inefficiency Detection with Prism Ops X-FIT
+++++++++++++++++++++++++++++++++++++++++++

Prism Ops uses X-FIT machine learning to detect and monitor the behaviors of VMs running within the managed clusters.

Using machine learning Prism Ops analyzes the data, and applies a classification to VMs that are inefficient. The following are short descriptions of the different classifications:

  * **Overprovisioned:** VMs identified as using minimal amounts of assigned resources.
  * **Inactive:** VMs that have been powered off for a period of time or that are running VMs that do not consume any CPU, memory, or I/O resources.
  * **Constrained:** VMs that could see improved performance with additional resources.
  * **Bully:** VMs identified as using an abundance of resources and affecting other VMs.

#. Select :fa:`bars` **> Dashboard** (if not already there).

#. From the Dashboard, take a look at the VM Efficiency widget. This widget gives a summary of inefficient VMs that Prism Ops’s X-FIT machine learning has detected in your environment. Click on the **View All Inefficient VMs** link at the bottom of the widget to take a closer look.

   .. figure:: images/ppro_58.png

#. You are now viewing the Efficiency focus. The *Efficiency Detail* column lists specifics on why Prism Ops flagged each VM as inefficient. You can hover the text in the *Efficiency Detail* column to view the full description.

   .. figure:: images/ppro_59.png

#. Once an admin has examined the list of VM on the efficiency list, they can determine any VMs they wish to take action against. VMs that have too many or too few resources will require the individual VMs to be resized. This can be done using any of the various methods listed below:

   * **Manually:** An admin edits the VM configuration via Prism (or vCenter for ESXi) and changes the assigned resources.
   * **X-Play:** Use X-Play's automated playbooks to resize VM(s) automatically via a trigger or admin's direction. There will be a lab story example of this later in this lab.
   * **Automation:** Use some other method of automation such as powershell or REST-API to resize a VM.


   Using this machine learning data, Prism Ops is also able to generate baselines, or expected ranges, for VM, Host and Cluster metric data. The X-FIT algorithms learn the normal behavior of these entities and represent that as a baseline range on the different charts. Whenever a metric value deviates from this expected range, Prism Ops will raise an anomaly.

#. Now let's take a take a look at a VM by searching for and selecting ‘bootcamp_good_1’.

   .. figure:: images/ppro_61.png

#. Click on Metrics > CPU Usage. Notice a dark blue line, and a lighter blue area around it. The dark blue line is the CPU Usage. The light blue area is the expected CPU Usage range for this VM. This particular VM is running an application that is upgraded at the same time each day, which explains the usage pattern. Notice that X-FIT detects this usage pattern, and has adjusted the expected range automatically. In this case, an anomaly has been raised for this VM, because the CPU Usage is far above the expected range. You can also reduce the time range to *Last 24 hours* to examine the chart more closely.

   .. figure:: images/ppro_60.png

#. Click **Alert Settings** at the upper right to set an alert policy.

#. On the right hand side, you can change some of the configurations however you choose. In the below example, under *Behavioral Anomaly*, *Every time there is an anomaly, alert* was set to Warning, and *Ignore all anomalies between* was set to 10% and 70%. Under Static threshold, *Alert Critical if* was configured to > 95%.

   .. figure:: images/ppro_25.png

#. Hit **Cancel** to exit the policy creation workflow.

Automatically Increase Constrained VM Memory with X-Play
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Now let's look at how we can take automated action to resolve some of these inefficiencies. For this lab we will assume that this VM is constrained for memory, and will show how we can automatically remediate the right sizing of this VM. We will also use a custom ticket system to give an idea of how this typical workflow could integrate with ticketing system such as ServiceNow.

#. Navigate to your **_Initials_-LinuxToolsVM**. The examples will use a VM called **ABC - VM**.

   .. figure:: images/rs1.png

#. Note the current **Memory Capacity** of the VM, as we will later increase it with X-Play. The placement of widgets may differ for you compared to the below example. Also, you may need to scroll down within the **Properties** widget to find this value.

   .. figure:: images/rs2.png

#. Navigate to the **Action Gallery** using the search bar.

   .. figure:: images/rs3.png

#. Select the **REST API** action and choose the **Clone** operation from the actions menu.

   .. figure:: images/rs4.png

#. We are creating an Action that we can later use in our playbook to generate a service ticket. Fill in the following values, and then click **Copy**.

   - **Name:** *Initials* - Generate Service Ticket
   - **Method:** POST
   - **URL:** http://<GTSPrismOpsLabUtilityServer_IP_ADDRESS>/generate_ticket/
   - **Request Body:**

   ::

     ``{"vm_name":"{{trigger[0].source_entity_info.name}}","vm_id":"{{trigger[0].source_entity_info.uuid}}","alert_name":"{{trigger[0].alert_entity_info.name}}","alert_id":"{{trigger[0].alert_entity_info.uuid}}"}``

   - **Request Header:**

   ::

   Content-Type:application/json;charset=utf-8

   .. figure:: images/rs5.png

#. Use the search bar to navigate to **Playbooks**.

   .. figure:: images/rs6.png

#. Now we will create a Playbook to automate the generation of a service ticket. Click **Create Playbook** at the top of the table view.

   .. figure:: images/rs7.png

#. Select **Alert** as a trigger

   .. figure:: images/rs8.png

#. Search and select **VM {vm_name} Memory Constrained** as the alert policy.

   .. figure:: images/rs9.png

#. Select the *Specify VMs* radio button and choose the **_Initials_-LinuxToolsVM**. This will make it so only alerts raised on your VM will trigger this Playbook.

   .. figure:: images/rs10.png

#. First, we would like to generate a ticket for this alert. Click **Add Action** on the left side, and select the **_Initials_ - Generate Service Ticket** action. Note: For this lab, we set up our own ticketing system to illustrate the full workflow.  You may noticed there is a ServiceNow action which can achieve the same workflow, but specifically for ServiceNow.

   .. figure:: images/rs11.png

#. Notice the details from the **Generate Service Ticket** action are automatically filled in for you.

   .. figure:: images/rs12.png

#. Next we would like to notify someone that the ticket was created by X-Play. Click **Add Action** and select the Email action. Fill in the field in the email action. Here are the examples.

   ::

      Be sure to replace <GTSPrismOpsLabUtilityServer_IP_ADDRESS> in the message with its IP Address.

   - **Recipient:** - Fill in your email address.
   - **Subject :**

   ::

      ``Service Ticket Pending Approval: {{trigger[0].alert_entity_info.name}}``

   - **Message:**

   ::

     ``The alert {{trigger[0].alert_entity_info.name}} triggered Playbook {{playbook.playbook_name}} and has generated a Service ticket for the VM: {{trigger[0].source_entity_info.name}} which is now pending your approval. A ticket has been generated for you to take action on at http://<GTSPrismOpsLabUtilityServer_IP_ADDRESS>/ticketsystem``

   .. figure:: images/rs13.png

#. Click **Save & Close** button and save it with a name **_Initials_ - Generate Service Ticket for Constrained VM**. **Be sure to click _Enabled_ toggle.**

   .. figure:: images/rs14.png

#. Now we will create one more Playbook. This will be executed when we resolve the service ticket by adding memory to the affected VM, and sending an email once complete. Click **Create Playbook** at the top of the table view.

   .. figure:: images/rs15.png

#. Select **Manual** as the trigger. Note: The ticket system we have constructed for this lab will call the trigger API provided by manual trigger, however this API is not public. In 5.17, we are introducing a Webhook Trigger which will expose a public API that allows achieving this same behavior. Tools like ServiceNow can use this Webhook to call Prism Central to trigger a playbook.

   .. figure:: images/rs16.png

#. Select the **VM** entity type from the dropdown, as this playbook will be applied to VMs.

   .. figure:: images/rs17.png

#. Click **Add Action** on the left side and select the **VM Add Memory** action.

   .. figure:: images/rs18.png

#. Set the empty fields according to the screen below. Next, we would like to notify someone that an automated action was taken. Click **Add Action** to add the email action

   .. figure:: images/rs19.png

#. Fill in the field in the email action. Here are the examples.

   - **Recipient:** - Fill in your email address.
   - **Subject :** - ``Playbook {{playbook.playbook_name}} was executed.``
   - **Message:**``{{playbook.playbook_name}} has run and has added 1GiB of Memory to the VM {{trigger[0].source_entity_info.name}}.``

   .. note::

      You are welcome to compose your own subject message. The above is just an example. You could click on *Parameters* to further customize the message.

   .. figure:: images/rs20.png

#. Last, we would like to notify the ticket service to resolve the ticket. Click **Add Action** to add the *REST API* action. Fill in the following values replacing the <GTSPrismOpsLabUtilityServer_IP_ADDRESS> in the URL field.

   - **Method:** PUT
   - **URL:** http://<GTSPrismOpsLabUtilityServer_IP_ADDRESS>/resolve_ticket
   - **Request Body:** ``{"vm_id":"{{trigger[0].source_entity_info.uuid}}"}``
   - **Request Header:** Content-Type:application/json;charset=utf-8

   .. figure:: images/rs21.png

#. Click **Save & Close** button and save it with a name “*Initials* - Resolve Service Ticket”. **Be sure to click _Enabled_ toggle.**

   .. figure:: images/rs22.png

#. Now let's trigger the workflow. Navigate to the tab you opened at the beginning of the lab, with the **/alerts** URL [example 10.42.113.52/alerts]. Select the Radio for **VM Memory Constrained** and select your VM. Click the **Simulate Alert** button. This will simulate a memory constrained alert on your VM.

   .. figure:: images/rs23.png

#. You should receive an email to the email address you put down in the first playbook. It may take up to 5 minutes.

   .. figure:: images/rs24.png

#. Inside the email click the link to visit the ticket system. Alternatively, you can directly access the ticket system by navigating to http://`<GTSPrismOpsLabUtilityServer_IP_ADDRESS>`/ticketsystem from a new tab in your browser.

   .. figure:: images/rs25.png

#. Identify the ticket created for your VM, and click the vertical dots icon to show the Action menu. Click the **Run Playbook** option.

   .. figure:: images/rs26.png

#. Choose the 2nd playbook you created **_Initials_ - Resolve Service Ticket** and click **Submit**.

   .. figure:: images/rs27.png

#. Switch back to the previous tab with the Prism Central console open. Click on the **_Initials_ - Resolve Service Ticket** playbook, and then click the **Plays** tab towards the top of the view to take a look at the Plays that executed for this playbook. Click on the title of the Play in the table, to take a closer look.

   .. figure:: images/rs29.png

#. The sections in this view can be expanded to show more details for each item. If there were any errors, they would also be surfaced in this view.

   .. figure:: images/rs30.png

#. You can navigate back to your VM, and verify that the memory was indeed increased by 1 GiB.

   .. figure:: images/rs31.png

#. You should also get an email telling you that the playbook ran.

   .. figure:: images/rs32.png

Takeaways
.........

- Prism Ops is our solution to make IT OPS smarter and automated. It covers the IT OPS process, from intelligent detection to automated remediation.

- X-FIT is our machine learning engine to support smart IT OPS, including anomaly detection and inefficiency detection.

- X-Play - the IFTTT for the enterprise - is our engine to enable the automation of daily operations tasks.

- X-Play enables admins to confidently automate their daily tasks within minutes.

- X-Play is extensive, and can use a customer's existing APIs and scripts as part of its Playbooks, integrate with their existing ticketing workflows.
