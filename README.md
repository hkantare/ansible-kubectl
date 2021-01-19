# kubectl-role

This repository can be used to install kubectl on Virtual machine from Ansible-Galaxy Role **andrewrothstein.kubectl** .

These playbooks are intended to be a reference and starter's guide to building Ansible Playbooks for use with IBM Cloud and Schematics. These playbooks were tested on CentOS 7.x so we recommend that you use CentOS or RHEL to test these modules. In this deployment we install a kubectl on Vortual Machine.

Schematics Actions uses SSH to configure target VSIs on IBM Cloud. To ensure that all access to the target VSIs is secured it is assumed that SSH access is configured via a bastion host/jump server om IBM Cloud. This [IBM Cloud Automation repo](https://github.com/Cloud-Schematics) contains a number of example Terraform configs that deploy a VPC Gen 2 environment with bastion host access.

This playbook has been run and tested using VSIs in a VPC Gen2 environment, deployed using the IBM Cloud Multitier VPC Bastion LAMP example. To provision Multitier VPC Bastion LAMP on IBM cloud follow the steps [here](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)

## Prerequisites :

   - Ansible 1.2.9
   - [Multitier VPC Bastion LAMP](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)
   - SSH Key on IBM Cloud

## Running the playbook locally
 To run this example locally, create a hosts file in the root of the example folder and update it with the {target-host-ip}, {jump-server-ip} and respective ssh keys. Details of how to configure the host file manually can be found in the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups). 
 The kubectl can be installed using the following 
- commands:
    - ansible-galaxy role install roles/requirements.yml -p roles
    - ansible-playbook kubectlplaybook.yml -i hosts

## Running on IBM Cloud Schematics

IBM Cloud Schematics supports running Ansible playbooks natively. You can try the same example through various entry points. 

### Running Actions using the IBM Cloud Schematics API

1. Create a file containing the Schematics Action payload. The Action payload is a json object. Start with `{}` and add the required fields. 

    - Add basic information for action like `name`, `description`, the Schematics `location` you want the Action to be placed in, `resource_group` and tag for identification. Ensure that the Action `name` is unique.  

    ```
    "name": "Example-1",
    "description": "This Action install Kubectl on VSI using ansible",
    "location": "us-east",
    "resource_group": "Default",
    "tags": [
      "string"
    ]
    ```

    - Add the source repo for importing the Ansible playbooks. 
    ```
    "source_type": "GitHub", 
    "source": {
         "source_type" : "git",
         "git" : {
              "git_repo_url": "https://github.com/areddy548/kubectl-role"
         }
    }
    ```
    - Add the playbook which should run by default when `run` is triggered. Any playbook from the source repo can be selected. 
    ```
    "command_parameter": "kubectlplaybook.yml"
    ```

    - Add the Credentials required o run the configuration. For deploying code to VSI's these will be the SSH private keys for the bastion host and target VSIs. All credentials should be added as `name` and `value` and will be referred in the target section with the same name.
    ```
    "credentials": [
      {
        "name": "ssh_key",
        "value": "< SSH_KEY >",
        "metadata": {
          "type": "string",
          "default_value": "",
          "secure": true
        }
      }
    ]
    ```
    - Add Bastion host information. Refer to the credentials provided in the above step in `cred_ref`.
    ```
    "bastion_ref": {
      "name": "bastionhost",
      "type": "string",
      "description": "string",
      "resource_query": "< BASITION_HOST_IP_ADDRESS >",
      "credential_ref": "ssh_key"
    }
    ```
    - Add inventory information. Refer the credentials provided in above step in `cred_ref`. Multiple groups with multiple host can be provided.The following inventory file 
    ```
    [webserverhost]
    FIRST_WEB_SERVER_IP_ADDRESS
    SECOND_WEB_SERVER_IP_ADDRESS
    ```
    can be represented as 
    ```
    "targets_ref": [
      {
        "name": "webserverhost",
        "description": "Group of webservers",
        "credential_ref": "ssh_key",
        "bastion_ref": "bastionhost",
        "target_resources": [
          {
            "resource_id": "< FIRST_WEB_SERVER_IP_ADDRESS >"
          },
           {
            "resource_id": "< SECOND_WEB_SERVER_IP_ADDRESS >"
          }
        ]
      }
    ]
    ```

2. Create action by making a http request.
    - Headers: 
    Authorization : < Bearer ...>
    - Method: POST
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/actions`


3. Note the `ID` from step 1 and check the status of action by making http request. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/actions/<ID>`

4. Verify if the action is in `normal` state. 
    ```
    "state": {  
        "status_code": "normal",
        "status_message": "Action is normal and ready for execution"
    }
    ```
5. Create Job with a http request. Modify the payload with the `ID` received in step 1. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/jobs`

    ```
    {
        "command_object": "action",
        "command_object_id": "< ACTION_ID >",
        "command_name": "ansible_playbook_run"
    }
    ```

6. Check logs with a http request. Use the `ID` from step 4. 
    - Headers: 
    Authorization : < Bearer ...>
    - Method: GET
    - ENDPOINT: `https://schematics.cloud.ibm.com/v2/jobs/<JOB-ID>/logs`

## Outputs

Check the job logs for of TASK: `Display Index page content`. The content should give information about installation successful if everything completed successfully.

### Running Actions using the IBM Cloud Schematics UI

1. Create an Action by Providing *name*, *description*, *location* and *resource_group* from IBM Cloud Schematics.

2. Provide the Repo deatils in this case it is [repo](https://github.com/areddy548/kubectl-role) under **Ansible Playbook** Tab, if it is Private repo provide github token as **Personal access token**. Wait untill Action come to **Normal** state

3. Enter the VSI and bastion host details in **IBM Cloud Resource Invetory** Tab.

   ## Variables

      | Variable Name | Description |	Default Value |
      | ----- | ----- | ----- |
      | Bastion host IP | bastion floating IP address | |
      | IBM Cloud inventory IP addresses| VSI IP address | |
      | IBM Cloud resource inventory SSH key| SSH Private key used for VSI installation | |

4. Wait for Action to come into **Normal** state and click on **Run Action** Button to run the playbook

5. Check the logs by clicking on **Jobs** in side menu. Currently running job will be on top of the list, Once job come to **Run successful** status with out any error we can check results.

**Results** : 

Log into your VSI Instance by ssh to VSI using bastion, below is the command
```
ssh -i <ssh_privatekey_path> -o ProxyCommand="ssh -i <ssh_privatekey_path> -W %h:%p root@<bastion_ip>" root@<vsi_ip>
```

Check whether Kubectl is got installed or not.
