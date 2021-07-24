## Deploy_HA_Webapp_on_GCP 

This is the supporting documentation for deploying web application and DB on GCP instances.
Goal of this repository is to provide the details on replicateing the steps needed to deploy Web and DB instances on GCP compute instances along with creation of Load Balancer.

## Google Cloud Platform Project

1. You will need to create a Google Cloud Platform Project as a first step.
Make sure you are logged in to your Google Account (gmail, Google+, etc) and
point your browser to https://console.cloud.google.com/projectselector/compute/instances. You should see a page asking you to create your first Project.

1. When creating a Project, you will see a pop-up dialog box. You can specify custom names but the *Project ID* is globally unique across all Google Cloud Platform customers.

1. It's OK to create a Project first, but you will need to set up billing before you can create any virtual machines with Compute Engine. Find the menu icon at the top left, then look for the *Billing* link in the navigation bar.

1. In order for `ansible` to create Compute Engine instances, you'll need a 
[Service Account](https://cloud.google.com/compute/docs/access/service-accounts#serviceaccount). 
It's recommended that you create a new Service Account (don't use the default).
Make sure to create a new JSON formatted private key file for this Service Account.

1. Next you will want to install the
[Cloud SDK](https://cloud.google.com/sdk/) and make sure you've successfully authenticated and set your default project as instructed. I set this up in ansible-control node hosted on the same GCP project. 

1. You will also need to make sure and set up SSH keys that will allow you to
access your Compute Engine instances. You can either
[manually generate the keys](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys#createsshkeys) and
paste the public key into the [metadata server](https://console.cloud.google.com/compute/metadata/sshKeys)
or you can use `gcloud compute ssh` to access an existing Compute Engine instance
and it will handle generating the keys and uploading them to the metadata
server. For this demo, it is assumed you have opted to use
`gcloud compute ssh` and your private key is located at `$HOME/.ssh/google_compute_engine`.

## Software

1. Install Dependencies.  On Debian-7, you may run the following to install them.

    ```
    sudo apt-get install -y build-essential git python-dev python-pip
    ```
1. Install Ansible with the
[running from source instructions](http://docs.ansible.com/intro_installation.html#running-from-source).

1. Install GCP module dependencies

    ```
    pip install googleauth requests
    ```

1. For the purposes of the demo, you can set a couple of environment variables
to simplify your commands and SSH interactions.

    ```
    export ANSIBLE_HOST_KEY_CHECKING=False
    ```

## Ansible setup

1. Check out this repository so that you can use pre-canned configuration
and demo files.
    ```
    cd $HOME
    git clone https://github.com/arjipurna/Deploy_HA_Webapp_on_GCP.git 
    ```

1. Edit the `gce_vars/auth` file and specify your Project ID in the
`project_id` variable, Service Account email address in the `service_account_email` variable,
and the location of your JSON key (downloaded earlier) in the `credentials_file` variable.
    ```
    ---
    # Google Compute Engine required authentication global variables
    # (Replace 'YOUR_PROJECT_ID' with the Project ID used in creating your GCP project.)
    project: YOUR_PROJECT_ID
    service_account_file: /path/to/your/json_key_file
    ```
Also, when creating a DB, MySql-DB creation role uses vars which is encrypted using ansible-vault. I have stored the passwroed under a file in my home directory and passing it when running the playbook. 
    ```
    # cat ~/.vault_pwd
    # password to be stored under the file "test123"
    ```

# Run time!

You've now completed all of the necessary setup to replicate the steps. Now, you'll use `ansible-playbook` to create and bootstrap the instances, install DB, Create DB, DB User, install webapp by cloing git repo, install Apache, and set up a Compute Engine load-balancer.

## Run the playbook

Use the `ansible-playbook` command to create the instances based on the attributes in the configuration files. For the four instances, this should take roughly 2 minutes to create the new Compute Engine instances

```
ansible-playbook -v -i ansible_hosts site.yml -e 'ansible_python_interpreter=/usr/bin/python3' --private-key=~/.ssh/google_compute_engine --vault-password-file ~/.vault_pwd
```

1. The output from this command will display the public IP address associated
with your new load-balancer. You can also look in the Developers Console
under *Networking &gt; Load balancing* and find the Forwarding Rules under the 
"Advanced" menu.

1. Ok, let's test it out! Put the public IP address of your load-balancer into
your browser and take a look at the result. Within a few seconds you should
start to see a flicker of pages that will randomly bounce across each of your
instances.

    For the demo, a javascript function is set to fire when the page loads
that pauses for a half-second, and then reloads itself. Since we installed
a modified Apache configuration file to disable client-side caching *and* we
enabled Apache's `mod_headers`, each "reload" results in a new HTTP request
to the page. This is just a fancy hands-free way of asking you to do a
"hard refresh" of the load-balancer IP address in order to see the cycling
between instances.

# All done!

That's it for the demo. There is a lot of other functionality for Compute Engine in Ansible.
