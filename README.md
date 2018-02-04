# qwiklabs-gcp-essentials-ansible
Ansible automation for the Qwiklabs GCE Essentials Quest

## Just for kicks
This repo is intended to demonstrate the art of the possible with GCP and Ansible. Obviously, just running these playbooks defeats the purpose of the Qwiklabs training. So think of this automation as extra credit.

## Incomplete

The GCP Ansible modules are still in preview. This means the interfaces may change. Some endpoints are not yet automatable with Ansible, like Google Cloud Launcher. Those exercises are skipped here.

## Setting up your environment

### virtualenv

First, set up a Python virtual environment.

RHEL, CentOS:

```
sudo su -
yum install -y python-pip python-virtualenv
virtualenv --system-site-packages ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto boto
```

Fedora:

```
sudo su -
dnf install -y python-pip python2-virtualenv
virtualenv --system-site-packages ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto boto
```

Debian, Ubuntu:

```
sudo su -
apt-get update
apt-get install -y python-virtualenv python-pip
virtualenv --system-site-packages ENV
source ENV/bin/activate
pip install ansible apache-libcloud jmespath PyCrypto boto
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

### Create your Google Storage interoperable access keys

In the GCP Console, go to Storage > Settings > Interoperability. Enable interoperable access and create new keys. Add them to ```gs_keys.json``` which is git ignored.

```
{
 "gs_access_key": "foofoofoo",
 "gs_secret_key": "barbarbar"
}
```

### Get the dynamic inventory script

Download ```gce.py``` and ```gce.ini``` into your inventory directory.

```
mkdir inventory
curl -o inventory/gce.py https://raw.githubusercontent.com/ansible/ansible/stable-2.4/contrib/inventory/gce.py
curl -o inventory/gce.ini https://raw.githubusercontent.com/ansible/ansible/stable-2.4/contrib/inventory/gce.ini
```

### Install gcloud SDK

If you haven't already, install the gcloud SDK,  https://cloud.google.com/sdk/downloads, and run the initial set up:

```
source ~/.bashrc
gcloud init
```

Select your project for this exercise and the appropriate zone, which in this quest is ```us-central1-c```.

If you haven't configured non-root access to the Docker socket, then you'll have to ```glcoud init``` from root as well.

Note: I also had to add a symlink to bin since the shell module doesn't source ```.bashrc```.

```
sudo su -
cd /bin
ln -s /home/jason/Downloads/google-cloud-sdk/bin/gcloud
```

## Running the playbooks

Don't forget to update your absolute paths. For example, this repo has lots of instances of ```/home/jason/qwiklabs-gcp-essentials-ansible```. Change that so it's appropriate for your deployment.

```
ansible-playbook -i inventory/gce.py 01\ Creating\ a\ Virtual\ Machine.yml
ansible-playbook 02\ Getting\ Stargetd\ with\ Cloud\ Shell\ \&\ gcloud.yml
ansible-playbook -i inventory/gce.py 04\ Creating\ a\ Persistent\ Disk.yml
```
