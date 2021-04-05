# ansible-kubectl

Ansible playbook for installing kubectl on Virtual machine from Ansible-Galaxy Role **andrewrothstein.kubectl**.

## About this playbook

These playbooks are intended to be a reference and starter's guide to building Ansible Playbooks for use with IBM Cloud and Schematics. These playbooks were tested on CentOS 7.x so we recommend that you use CentOS or RHEL to test these modules.

## Prerequisites:

   - SSH Key on IBM Cloud

## Input variables

|Input variable|Required/ optional|Data type|Description|
|--|--|--|--|
|Bastion_IP|Required|String|bastion floating IP address|
|VSI_IP|Required|String|IBM Cloud VSI IP address|
|SSH_KEY|Required|String|IBM Cloud SSH Private key used for VSI installation|

## Running the playbook in Schematics by using UI

1. Open the [Schematics action configuration page](https://cloud.ibm.com/schematics/actions/create?name=kubectle&url=https://github.com/Cloud-Schematics/ansible-kubectl).
2. Review the name for your action, and the resource group and region where you want to create the action. Then, click **Create**.
3. Select the `site.yml` playbook.
4. Select the **Verbosity** level to control the depth of information that will be shown when you run the playbook in Schematics.
5. Expand the **Advanced options**.
6. Enter all required input variables as key-value pairs. Then, click **Next**.
7. Click **Check action** to verify your action details. The **Jobs** page opens automatically. You can view the results of this check by looking at the logs.
8. Click **Run action** to deploy the kubectl. You can monitor the progress of this action by reviewing the logs on the **Jobs** page.

## Running the playbook in Schematics by using the command line

 1. Create the Schematics action. Enter all the input variable values that you retrieved earlier. When you run this command and are prompted to enter a GitHub token, enter the return key to skip this prompt.
   ```
   ibmcloud schematics action create --name kubectl --location us-south --resource-group default --template https://github.com/Cloud-Schematics/ansible-kubectl --playbook-name site.yml --input "mysql_port": "<mysql_port>" --input "httpd_port": "<httpd_port>" --input "dbuser": "<dbuser>" --input "upassword": "<db_password>"
   ```

   Example output:
   ```
   Enter github-token>
   The given --inputs option region: is not correctly specified. Must be a variable name and value separated by an equals sign, like --inputs key=value.

   ID               us-south.ACTION.kubectl.1aa11a1a
   Name             kubectl
   Description
   Resource Group   default
   user State       live

   OK
   ```

2. Verify that your Schematics action is created and note the ID that was assigned to your action.
   ```
   ibmcloud schematics action list
   ```

3. Create a job to run a check for your action. Replace `<action_ID>` with the action ID that you retrieved. In your CLI output, note the **ID** that was assigned to your job.
   ```
   ibmcloud schematics job create --command-object action --command-object-id <action_ID> --command-name ansible_playbook_check
   ```

   Example output:
   ```
   ID                  us-south.JOB.kubectl.fedd2fab
   Command Object      action
   Command Object ID   us-south.ACTION.kubectl.1aa11a1a
   Command Name        ansible_playbook_check
   Name                JOB.kubectl.ansible_playbook_check.2
   Resource Group      a1a12aaad12b123bbd1d12ab1a123ca1
   ```

4. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>
   ```

5. Create another job to run the action. Replace `<action_ID>` with your action ID.
   ```
   ibmcloud schematics job create --command-object action --command-object-id <action_ID> --command-name ansible_playbook_run
   ```

6. Verify that your job ran successfully by retrieving the logs.
   ```
   ibmcloud schematics job logs --id <job_ID>

## Verification

Check the job logs for of TASK: `Display Index page content`. The content should give information about installation successful if everything completed successfully.

Log into your VSI Instance by ssh to VSI using bastion, below is the command
```
ssh -i <ssh_privatekey_path> -o ProxyCommand="ssh -i <ssh_privatekey_path> -W %h:%p root@<bastion_ip>" root@<vsi_ip>
```

Check whether Kubectl is installed or not.