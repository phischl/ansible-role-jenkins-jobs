# Ansible role for managing jenkins jobs

This is a role to manage jenkins jobs

## Requirements

* Debian/Ubuntu

## Vars

| Name                         | Description                                                             | Default               | Required |
|------------------------------|-------------------------------------------------------------------------|-----------------------|----------|
| jenkins_jobs_server_url      | URL to Jenkins server                                                   | http://localhost:8080 | yes      | 
| jenkins_jobs_server_user     | Username for connection to server                                       | /                     | yes      |
| jenkins_jobs_server_password | **API token** for user authentication                                   | /                     | yes      |
| jenkins_jobs_credentials     | List of credentials to manage (see variable `jenkins_jobs_credentials`) | []                    | no       |
| jenkins_jobs_list            | List of jobs to manage  (see variable `jenkins_jobs_list`               | []                    | no       |

### Var jenkins_jobs_credentials
If `ssh_private_key` is set the credential will be created as `SSHUserPrivateKeyBinding`.  
Else it will be created as `UsernamePasswordMultiBinding`.

| Name            | Description                          | Default | Required |
|-----------------|--------------------------------------|---------|----------|
| id              | ID of the secrets for usage in jobs  | /       | yes      | 
| username        | A username for the secret            | /       | yes      |
| password        | A password for the secret.           | /       | yes      |
| ssh_private_key | An SSH private key.                  | []      | no       |

```yaml
    jenkins_jobs_credentials:
      - id: "secret_userpass"
        username: "my_user"
        password: "supersecret"
      - id: "my_ssh_key"
        username: "my_ssh_user"
        password: "my_ssh_pass"
        ssh_private_key: |
          -----BEGIN OPENSSH PRIVATE KEY-----
          ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789=
          -----END OPENSSH PRIVATE KEY-----
      - id: "my_ssh_key_without_pass"
        username: "my_ssh_user"
        ssh_private_key: |
          -----BEGIN OPENSSH PRIVATE KEY-----
          ABCDEFGHIJKLMNOPQRSTUVWXYZ9876543210=
          -----END OPENSSH PRIVATE KEY-----
```

### Var jenkins_jobs_list
| Name           | Description                                                                  | Default | Required |
|----------------|------------------------------------------------------------------------------|---------|----------|
| name           | Name of the job                                                              | /       | yes      | 
| type           | Can be `folder` or `job`                                                     | job     | yes      |
| scm            | config for Git repo to checkout (see variable `jenkins_jobs_list.scm`).      | /       | no       |
| build_wrappers | List of build wrappers (see variable `jenkins_jobs_list.build_wrappers`)     | []      | no       |
| parameters     | List of parameters for the job (see variable `jenkins_jobs_list.parameters`) | []      | no       |
| builders       | List of builders for the job (see variable `jenkins_jobs_list.builders`)     | []      | no       |
| schedule       | Schdule job execution in cron format                                         | /       | no       |
| output_colored | Enable colored output for the job (requires ansicolor plugin)                | false   | no       |

**Example**
```yaml
jenkins_jobs_list:
  - name: "test"
    type: folder
  - name: "test/foo"
    type: "job"
    schedule: "*/5 * * * *"
    builders:
      - type: shell
        command: echo "hello world"
```

### Var jenkins_jobs_list.scm
| Name           | Description                                                                                      | Default | Required |
|----------------|--------------------------------------------------------------------------------------------------|---------|----------|
| url            | GIT Url                                                                                          | /       | yes      | 
| credentials_id | ID of SSH credentials which should be used (see e.g. `jenkins_jobs_credentials[].credentials_id` | /       | yes      |
| ref_spec       | Refspec to clone/checkout                                                                        | */main  | no       |

**Example**
```yaml
jenkins_jobs_list:
  - name: "foo"
    scm:
      url: "git@gitlab.local:foobar/path/to/repo.git"
      credentials_id: "ssh"
      ref_spec: "*/dev"
  ## ...
```

### Var jenkins_jobs_list.build_wrappers (`type: ssh-agent`)
| Name           | Description                                                                                  | Default | Required |
|----------------|----------------------------------------------------------------------------------------------|---------|----------|
| type           | Type of build wrapper. Can be `ssh-agent`                                                    | /       | yes      | 
| credentials_id | ID of credentials which should be used (see e.g. `jenkins_jobs_credentials[].credentials_id` | /       | yes      |

**Example**
```yaml
build_wrappers:
  # Setup SSH agent
  - type: ssh-agent
    credentials_id: "ssh"
```

### Var jenkins_jobs_list.build_wrappers (`type: secret`)
| Name             | Description                                                                                              | Default | Required |
|------------------|----------------------------------------------------------------------------------------------------------|---------|----------|
| type             | Type `secret`                                                                                            | /       | yes      | 
| credentials_id   | ID of credentials which should be used (see e.g. `jenkins_jobs_credentials[].credentials_id`             | /       | yes      |
| username_var     | Name of env var to store the username to                                                                 | /       | no       |
| password_var     | Name of env var to store the password to                                                                 | /       | no       |
| ssh_key_file_var | Name of env var to store the filepath for the SSH private key to (can only be used with SSH key secrets) | /       | no       |

**Example**
```yaml
build_wrappers:
  # Get username/password secret
  - type: secret
    credentials_id: "ansible_vault"
    username_var: DEMO_MY_USER
    password_var: DEMO_MY_PASSWORD
  # Get SSH key secret
  - type: secret
    credentials_id: "crazy_key"
    username_var: DEMO_SSH_MY_USER
    password_var: DEMO_SSH_MY_PASSWORD
    ssh_key_file_var: DEMO_SSH_MY_KEY_FILE
```

### Var jenkins_jobs_list.parameters (`type: bool`)
| Name        | Description                         | Default | Required |
|-------------|-------------------------------------|---------|----------|
| type        | Type `bool`                         | /       | yes      | 
| name        | Name of the parameter               | /       | yes      |
| description | A description to display in Jenkins | ""      | no       |
| default     | Can be `true` or `false`            | false   | no       |

**Example**
```yaml
parameters:
  - name: REBOOT_ALLOWED
    type: bool
    default: true
```

### Var jenkins_jobs_list.parameters (`type: text`)
| Name        | Description                         | Default | Required |
|-------------|-------------------------------------|---------|----------|
| type        | Type `text`                         | /       | yes      | 
| name        | Name of the parameter               | /       | yes      |
| description | A description to display in Jenkins | ""      | no       |
| default     | A default value                     | ""      | no       |

**Example**
```yaml
parameters:
  - name: SERVICE_NAME
    type: text
    default: apache2
```

### Var jenkins_jobs_list.parameters (`type: choice`)
| Name        | Description                                                | Default | Required |
|-------------|------------------------------------------------------------|---------|----------|
| type        | Type `choice`                                              | /       | yes      | 
| name        | Name of the parameter                                      | /       | yes      |
| description | A description to display in Jenkins                        | ""      | no       |
| choices     | List with possible choices (`["blue", "green", "yellow"]`) | /       | yes      |

**Example**
```yaml
parameters:
  - name: SERVER
    type: choice
    choices:
      - foo.local
      - bar.local
```

### Var jenkins_jobs_list.builders (`type: shell`)
| Name        | Description                                                | Default | Required |
|-------------|------------------------------------------------------------|---------|----------|
| type        | Type `shell`                                               | /       | yes      | 
| command     | Shell command to execute                                   | /       | yes      |

**Example**
```yaml
builders:
  - type: shell
    command: echo "hello world"
```


## Full playbook example
```yaml
---
- name: Manage Jenkins jobs
  hosts: localhost
  gather_facts: false
  vars:
    _all_managed_hosts: "{{ hostvars.keys() | sort }}"
    jenkins_jobs_credentials:
      - id: "ssh_key_for_git"
        username: "my_ssh_user_for_git"
        ssh_private_key: |
          -----BEGIN OPENSSH PRIVATE KEY-----
          ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789=
          -----END OPENSSH PRIVATE KEY-----
      - id: "ssh_key_for_ssh"
        username: "my_ssh_user_for_ssh"
        ssh_private_key: |
          -----BEGIN OPENSSH PRIVATE KEY-----
          ABCDEFGHIJKLMNOPQRSTUVWXYZ9876543210=
          -----END OPENSSH PRIVATE KEY-----
      - id: "decrypt_pass"
        password: "my_secret_password"
    jenkins_jobs_list:
      - name: "demo"
        type: folder
      - name: "demo/my_job"
        scm:
          url: "git@gitlab.local:foobar/path/to/repo.git"
          credentials_id: "ssh_key_for_git"
          ref_spec: "*/master"
        build_wrappers:
          - type: ssh-agent
            credentials_id: "ssh_key_for_ssh"
          - type: secret
            credentials_id: "decrypt_pass"
            password_var: DEMO_MY_PASSWORD
          - type: secret
            credentials_id: "ssh_key_for_ssh"
            username_var: DEMO_SSH_MY_USER
            ssh_key_file_var: DEMO_SSH_MY_KEY_FILE
        parameters:
          - name: SERVER
            type: choice
            choices: ["foo.local", "bar.local"]
          - name: ALLOW_REBOOT
            type: bool
          - name: ECHO_MESSAGE_BEFORE_REBOOT
            type: text
            default: "The server will reboot"
        builders:
          - type: shell
            command: |
              ssh ${SERVER} "echo '${ECHO_MESSAGE_BEFORE_REBOOT}'"
              if [ "${ALLOW_REBOOT}" == "true" ]; then
                ssh ${SERVER} "echo '${ECHO_MESSAGE_BEFORE_REBOOT}'"
                ssh ${SERVER} "reboot"
              fi
  roles:
    - role: jenkins_jobs
      vars:
        jenkins_jobs_server_url: "http://jenkins.local:8080"
        jenkins_jobs_server_user: "{{ __vault_credentials.jenkins_api_user }}"
        jenkins_jobs_server_password: "{{ __vault_credentials.jenkins_api_token }}"
```
