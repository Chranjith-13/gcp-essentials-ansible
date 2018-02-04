# qwiklabs-gcp-essentials-ansible
Ansible automation for the Qwiklabs GCE Essentials Quest

## Just for kicks
This repo is intended to demonstrate the art of the possible with GCP and Ansible. Obviously, just running these playbooks defeats the purpose of the Qwiklabs training. So think of this automation as extra credit.

## Setting up your environment

### virtualenv

First, set up a Python virtual environment.

RHEL, CentOS:

```
sudo su -
yum install python-pip python-virtualenv
virtualenv ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto
```

Fedora:

```
sudo su -
dnf install python-pip python2-virtualenv
virtualenv ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto
```

Debian, Ubuntu:

```
sudo su -
apt-get update
apt-get install -y python-virtualenv python-pip
virtualenv ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto
```

### GCP Service Account

Create a service account according to the docs at https://developers.google.com/identity/protocols/OAuth2ServiceAccount#creatinganaccount.

Name the account ```ansible```. From the Cloud Shell, replacing ```one-two-345678``` with your project id:

```
gcloud projects add-iam-policy-binding one-two-345678 --member serviceAccount:ansible@one-two-345678.iam.gserviceaccount.com --role roles/editor
```

### Download the JSON credentials

Following the instructions at https://support.google.com/cloud/answer/6158849?hl=en&ref_topic=6262490#serviceaccounts, download the JSON credentials. Place them in this directory, but be sure to name them ```creds.json``` so they're ignored by .gitignore.

```
cp ~/Downloads/name_of_credentials.json creds.json
chmod 600 creds.json
```

### Save you ssh private key

From your Cloud Shell, copy ```.ssh/google_compute_engine``` to a file of the same name in this project's root directory. Be sure to name it ```google_compute_engine``` so it's ignored by .gitignore.

Then instantiate ssh-agent and add the key.

```
chmod 600 google_compute_engine
ssh-agent bash
ssh-add google_compute_engine
```

### Get the dynamic inventory script

Download ```gce.py``` and ```gce.ini``` into your inventory directory.

```
mkdir inventory
curl -o inventory/gce.py https://raw.githubusercontent.com/ansible/ansible/stable-2.4/contrib/inventory/gce.py
curl -o inventory/gce.ini https://raw.githubusercontent.com/ansible/ansible/stable-2.4/contrib/inventory/gce.ini
```

## Running the playbooks

Don't forget to update your absolute paths. For example, this repo has lots of instances of ```/home/jason/qwiklabs-gcp-essentials-ansible```. Change that so it's appropriate for your deployment.

```
ansible-playbook -i inventory/gce.py 01\ Creating\ a\ Virtual\ Machine.yml
```