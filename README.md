# ansible-kubectl

Ansible playbook for installing kubectl on Virtual machine from Ansible-Galaxy Role **andrewrothstein.kubectl** .

These playbooks are intended to be a reference and starter's guide to building Ansible Playbooks for use with IBM Cloud and Schematics. These playbooks were tested on CentOS 7.x so we recommend that you use CentOS or RHEL to test these modules.

Schematics Actions uses SSH to configure target VSI's on IBM Cloud. To ensure that all access to the target VSI's is secured it is assumed that SSH access is configured via a bastion host/jump server om IBM Cloud. This [IBM Cloud Automation repo](https://github.com/Cloud-Schematics) contains a number of example Terraform configs that deploy a VPC Gen 2 environment with bastion host access.

This playbook has been run and tested using VSI's in a VPC Gen2 environment, deployed using the IBM Cloud Multitier VPC Bastion LAMP example. To provision Multitier VPC Bastion LAMP on IBM cloud follow the steps [here](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)

You can create a Schematics Action, using these playbooks; and allow your team members to perform these Actions in a controller manner.
Follow the instruction to onboard these Ansible playbooks as Schematics Action, and run them as Schematics Jobs.

## Prerequisites:
   - Ansible 1.2.9
   - [Multitier VPC Bastion LAMP](https://github.com/Cloud-Schematics/multitier-bastion-vpc-lamp)
   - SSH Key on IBM Cloud

## Inputs:
  - bastion floating IP address
  - IBM Cloud VSI IP address
  - IBM Cloud SSH Private key used for VSI installation

## Run the ansible playbook using Ansible Playbook command
 To run this example locally, create a hosts file in the root of the example folder and update it with the {target-host-ip}, {jump-server-ip} and respective ssh keys. Details of how to configure the host file manually can be found in the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups). 
 The kubectl can be installed using the following 
- commands:
    - ansible-galaxy role install roles/requirements.yml -p roles
    - ansible-playbook kubectl.yml -i hosts

## Run the ansible playbook using Schematics API    

- Create a Schematics action: "kubectl-role"
   Use the `POST {url}/v2/actions` with the following payload:
   Url: https://schematics.cloud.ibm.com

   Pass header: Authorization: {bearer token}

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
              "git_repo_url": "https://github.com/Cloud-Schematics/ansible-kubectl
         }
    }
    ```
    - Add the playbook which should run by default when `run` is triggered. Any playbook from the source repo can be selected. 
    ```
    "command_parameter": "kubectl.yml"
    ```

    - Add the Credentials required o run the configuration. For deploying code to VSI's these will be the SSH private keys for the bastion host and target VSI's. All credentials should be added as `name` and `value` and will be referred in the target section with the same name.
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
    "bastion": {
      "name": "bastionhost",
      "type": "string",
      "description": "string",
      "resource_query": "< BASITION_HOST_IP_ADDRESS >",
      "credential_ref": "ssh_key"
    }
    ```
    - Add inventory information. 
    ```
    "targets_ini": "[webserverhost]
    FIRST_WEB_SERVER_IP_ADDRESS
    SECOND_WEB_SERVER_IP_ADDRESS"
    
    ```

- Create & run the Schematics Job for "kubectl-role"

  Use the `POST {url}/v2/jobs` with the following payload:
  Url: https://schematics.cloud.ibm.com
  
  Pass header: Authorization: {bearer token}
 
    ```
    {
      "command_object": "action",
      "command_object_id": {action-id from the response of above request},
      "command_name": "ansible_playbook_run",
      "command_parameter": "kubectl.yml"
    }
    ```

- Check the Schematics Job status and the ansible logs:

  Use the `GET {url}/v2/jobs/{job-id}/logs`. 
  Url: https://schematics.cloud.ibm.com
  
  Pass header: Authorization: {bearer token}


## Outputs

Check the job logs for of TASK: `Display Index page content`. The content should give information about installation successful if everything completed successfully.

### Running Actions using the IBM Cloud Schematics UI

- Open https://cloud.ibm.com/schematics/actions to view the list of Schematics Actions.
- Click `Create action` button to create a new Schematics Action definition.
- In the Create action page - section 1, provide the following inputs, to create a `kubectl-role` action in `Draft` state.
  * Action name : kubectl-role
  * Resource group: default
  * Location : us-east
- In the Create action page - section 2, provide the following input
  * Github url : https://github.com/Cloud-Schematics/ansible-kubectl
  * Click on `Retrieve playbooks` button
  * Select `kubectl.yml` from the dropdown
- In the Create action page - Advanced options, provide the following input
  * Add `BASITION_HOST_IP_ADDRESS` as key and `<Bastion floating IP address>` as value
  * Add `VSI_IP_ADDRESS` as key and `<VSI IP address>` as value
  * Add `SSH_KEY` as key and `<SSH Private key>` as value
- Press the `Next` button, and wait for the newly created `kubectl-role` action to move to `Normal` state.
- Once the `kubectl-role` action is in `Normal` state, you can press the `Run action` button to initiate the Schematics Job
  * You can view the job status and the job logs (or Ansible logs) in the Jobs page of the `kubectl-role` Schematics Action
  * Jobs page of the `kubectl-role` Schematics Action will list all the historical jobs that was executed using this Action definition

**Results** : 

Log into your VSI Instance by ssh to VSI using bastion, below is the command
```
ssh -i <ssh_privatekey_path> -o ProxyCommand="ssh -i <ssh_privatekey_path> -W %h:%p root@<bastion_ip>" root@<vsi_ip>
```

Check whether Kubectl is installed or not.
