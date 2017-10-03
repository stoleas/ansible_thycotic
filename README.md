# Ansible/Thycotic Integration

Various integrations for Thycotic Secret Server (EPM) with Ansible.

## Playbook Layout
### Directory Tree
```bash
ansible_thycotic/
├── group_vars
│   └── servers
│       └── vars.yml
├── hosts.ini
├── notes
│   ├── example_return_from_dowloadFileAttachment.txt
│   └── example_return_from_getSecret.txt
├── README.md
├── roles
│   ├── downloadFileAttachment
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── main.yml
│   │       └── soap_requests -> ../../../soap_requests/
│   ├── getSecret
│   │   ├── defaults
│   │   │   └── main.yml
│   │   └── tasks
│   │       ├── main.yml
│   │       └── soap_requests -> ../../../soap_requests/
│   ├── getToken
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   ├── main.yml
│   │   │   └── soap_requests -> ../../../soap_requests/
│   │   └── vars
│   │       └── Authenticate.yml
│   ├── updateSecret
│   │   └── tasks
│   │       ├── main.yml
│   │       └── soap_requests -> ../../../soap_requests/
│   └── uploadFileAttachment
│       └── tasks
│           ├── main.yml
│           └── soap_requests -> ../../../soap_requests/
├── site.yml
├── soap_requests
│   ├── Authenticate.xml
│   ├── DownloadFileAttachment.xml
│   ├── GetSecret.xml
│   ├── UpdateSecret.xml
│   └── UploadFileAttachment.xml
└── vault_pass.txt

24 directories, 21 files
```

## Playbook Role Definition
### getToken

getToken role should be invoked as an initialization function. This role will need to be ran before other Thycotic integrations due to Thycotic needing a token be set for further SOAP API invocations. This role calls the GetToken soap_request under ./soap_requests/GetToken.xml as a template.

##### Required Arguements/Variables #####
- req_username: This is the request username. This user should have the ability to access the secrets from the Tycotic Secret Server in successive calls of the Ansible/Thycotic integration.
- req_password: This is the request password corresponding to the req_username variable. This password should likely be put into Ansible Vault. In the demonstration in this variable is set both in the roles default/main.yml, and vars/Authenticate.yml, however due to include_vars, vars/Authenticate.yml overrides with an Ansible Vaulted file. For more instructions on how to use Vault visit: http://docs.ansible.com/ansible/latest/playbooks_vault.html
- epm_server_soap_endpoint: EPM server URL for SOAP requests. You should add the "/webservices/sswebservice.asmx" context path to the base path of your EPM Secret Server's URL.

##### Output Variables #####
- cur_user_token: This is the current authenticated user token. This variable should be passed to your other role's templates file as to not require manual intervention. This will also allow you to authenticate to other SOAP API requests to the EPM server.

##### Available Tags #####
1. init
2. getToken

### getSecret

getSecret is currently a helper-role for updateSecret. This role can be expanded to provide secret information for other roles and plays in this playbook.

##### Required Arguements/Variables #####
- cur_user_token: Output variable from getToken role.
- secret_id: The Secret ID integer of the secret you want to query
- epm_server_soap_endpoint: EPM server URL for SOAP requests. You should add the "/webservices/sswebservice.asmx" context path to the base path of your EPM Secret Server's URL.

##### Output Variables #####
1. get_secret_field_name
2. get_secret_value
3. get_secret_folder_id
4. get_secret_secret_id
5. get_secret_field_id
6. get_secret_is_file
7. get_secret_is_notes
8. get_secret_is_password
9. get_secret_field_display_name

### updateSecret

Updates a secret with the corresponding value set as input.

##### Required Arguements/Variables #####
- cur_user_token: Output variable from getToken role.
- epm_server_soap_endpoint: EPM server URL for SOAP requests. You should add the "/webservices/sswebservice.asmx" context path to the base path of your EPM Secret Server's URL.
- secret_id: Secret ID of the secret you want to update.
- secret_field: The name of the secret you want to update (case-sensitvie)
- secret_value: The updated value you want to set for the secret.
- secret_name: The name of the secret you want to update.
- get_secret_field_name: Variable passed in through getSecret role.
- get_secret_value: Variable passed in through getSecret role.
- get_secret_folder_id: Variable passed in through getSecret role.
- get_secret_secret_id: Variable passed in through getSecret role.
- get_secret_field_id: Variable passed in through getSecret role.
- get_secret_is_file: Variable passed in through getSecret role.
- get_secret_is_notes: Variable passed in through getSecret role.
- get_secret_is_password: Variable passed in through getSecret role.
- get_secret_field_display_name: Variable passed in through getSecret role.

##### Output Variables #####
None

### downloadFileAttachment

Downloads a secret's file attachment by calling the DownloadFileAttachment SOAP API.

##### Required Arguements/Variables #####
- cur_user_token: Output variable from getToken role.
- epm_server_soap_endpoint: EPM server URL for SOAP requests. You should add the "/webservices/sswebservice.asmx" context path to the base path of your EPM Secret Server's URL.
- secret_id: The Secret ID of the secret you want to download the attached file from.

##### Output Variables #####
1. attachment_content: The contents of the attachment.
2. attachment_file_name: The file name of the attachment from EPM. A random key is appended to the file name to avoid file-name collisions.

### uploadFileAttachment

Uploads a secret file attachment to update the one currently in place on the EPM server.

##### Required Arguements/Variables #####
- cur_user_token: Output variable from getToken role.
- epm_server_soap_endpoint: EPM server URL for SOAP requests. You should add the "/webservices/sswebservice.asmx" context path to the base path of your EPM Secret Server's URL.
- secret_id: Secret ID of the secret you want to upload to.
- secret_value: The value of the file contents you want to upload.
- secret_file_name: The name of the file you want to upload.

##### Output Variables #####
None

## Needs Work/bugs
- uploadFileAttachment not handling multi-line file values.
- ansible_ssh_pass not accepting passsphrase as authentication method from add_hosts.

##### Output Variables #####
Author: Edward Quail
Email: equail@redhat.com