# purr-globus

## Reference Docs

[Argonne's guest collection documentation](https://www.alcf.anl.gov/support/user-guides/data-management/acdc/eagle-data-sharing/index.html) is great. I've asked, and they're fine with it being reused and adapted. It could be a template for "Sharing PURR Project Data using Globus".

(The link above was broken. The ALCF may have been moving documentation around.)

## Install instructions

https://docs.globus.org/globus-connect-server/v5.4/#install_section

For RHEL/CentOS 7

```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install https://downloads.globus.org/globus-connect-server/stable/installers/repo/rpm/globus-repo-latest.noarch.rpm

sudo yum install globus-connect-server54
```

## HTTPS

The collections here have HTTPS access enabled. HTTPS access still uses the Globus permissions. Guest collections can allow `public` anonymous access. Of course, the data access on the GCS instance will still follow the identity mappsing, so if a user creates a Guest Collection and makes a folder public, they can only do this for folders they already have access to.

### Example: Public Image

This random image is the folder `public` of the [Demo Time Globus Only](https://app.globus.org/file-manager?origin_id=79ced6ac-1141-4cc9-a8fe-b161e95eac2f&origin_path=%2F) guest collection. It should display automatically and can be download using `curl` or `wget`.

URL: `https://m-9b0939.ca528.03c0.data.globus.org/projectD/cursal-clinometry.png`

![Random Image](https://g-e77286.ca528.03c0.data.globus.org/public/dramaturgist-inlaw.png)

### Example: Private (Project-only) Image

This image is in `projectD` of the [mapped collection on top of `projfs`](https://app.globus.org/file-manager?origin_id=76062968-59fc-44aa-8633-3ca1e252be2b&origin_path=%2FprojectD%2F) (FUSE). You will need to authenticate to view it.

URL `https://m-9b0939.ca528.03c0.data.globus.org/projectD/cursal-clinometry.png`

<a href="https://m-9b0939.ca528.03c0.data.globus.org/projectD/cursal-clinometry.png" target="_blank">Click this to view image</a>

## HUBzero GCS Configuration for PURR

### EC2 Instance

```
AMI
us-west-2	jammy	22.04	amd64	hvm:ebs-ssd	20220901 ami-0c033eb565588ae0e	hvm

sudo hostnamectl set-hostname pgcs.stratus.jupyter-security.info
sudo apt-get update
sudo apt-get install -y emacs-nox fuse autofs
sudo apt-get -y upgrade
sudo reboot
```

### Install projfs

```
tar -zxf projfs.tar.gz
sudo cp projfs_source/projfs /sbin/mount.projfs
ls -l /sbin/mount.projfs 
-rwxr-x--- 1 root root 46904 Sep  9 03:24 /sbin/mount.projfs
```

### Create POSIX user accounts

Groups
```
for i in A B C D; do sudo groupadd pr-project$i; done
tail -4 /etc/group
pr-projectA:x:1001:
pr-projectB:x:1002:
pr-projectC:x:1003:
pr-projectD:x:1004:

sudo groupadd purr-users
```

`purr-data` POSIX user that will own the data
```
sudo useradd -M -s /sbin/nologin purr-data 
```

PURR user POSIX accounts
```
sudo useradd -m -G pr-projectA,pr-projectD,purr-users -s /bin/bash pmeunier
sudo useradd -m -G pr-projectB,pr-projectD,purr-users -s /bin/bash monawong
sudo useradd -m -G pr-projectC,pr-projectD,purr-users -s /bin/bash rpwagner

grep ^pr /etc/group
pr-projectA:x:1001:pmeunier
pr-projectB:x:1002:monawong
pr-projectC:x:1003:rpwagner
pr-projectD:x:1004:pmeunier,monawong,rpwagner

tail -4 /etc/passwd
purr-data:x:1005:
pmeunier:x:1006:
monawong:x:1007:
rpwagner:x:1008:
```

### Project directories

- Owned by `purr-data`
- Data in `/mnt/purr-data`
```
for i in A B C D
do
sudo mkdir -p /mnt/purr-data/projects/project$i
done
ls /mnt/purr-data/
projectA  projectB  projectC  projectD

sudo chown -R purr-data:purr-data /mnt/purr-data
sudo chmod -R go-rwx /mnt/purr-data
sudo mkdir -p /mnt/purr-fuse/projects
```

### Configure `projfs`

`/mnt/purr-fuse/projects/{project}` is mounted using autofs and FUSE from `/mnt/purr-data/projects/{project}`
- [/etc/auto.master.d/projfs.autofs](projfs.auto)
- [/etc/auto.purr-projects](auto.purr-projects)

```
sudo cp projfs.autofs  /etc/auto.master.d/projfs.autofs 
cat /etc/auto.master.d/projfs.autofs 
/mnt/purr-fuse /etc/auto.purr-projects

sudo cp auto.purr-projects /etc/auto.purr-projects 
cat /etc/auto.purr-projects 
projects    -fstype=projfs,source_user=purr-data,allow_other  :/mnt/purr-data/projects
sudo service autofs restart
```

```
sudo ls -lR /mnt
/mnt:
total 8
drwx------ 3 purr-data purr-data 4096 Sep  9 04:46 purr-data
drwxr-xr-x 3 root      root      4096 Sep  9 04:46 purr-fuse

/mnt/purr-data:
total 4
drwx------ 6 purr-data purr-data 4096 Sep  9 04:46 projects

/mnt/purr-data/projects:
total 16
drwx------ 2 purr-data purr-data 4096 Sep  9 04:46 projectA
drwx------ 2 purr-data purr-data 4096 Sep  9 04:46 projectB
drwx------ 2 purr-data purr-data 4096 Sep  9 04:46 projectC
drwx------ 2 purr-data purr-data 4096 Sep  9 04:46 projectD

/mnt/purr-data/projects/projectA:
total 0

/mnt/purr-data/projects/projectB:
total 0

/mnt/purr-data/projects/projectC:
total 0

/mnt/purr-data/projects/projectD:
total 0

/mnt/purr-fuse:
total 4
drwxr-xr-x 2 root root 4096 Sep  9 04:46 projects

/mnt/purr-fuse/projects:
total 0
```

Restart `autofs`
```
sudo service autofs start
sudo -u purr-data ls -R /mnt
/mnt:
purr-data  purr-fuse

/mnt/purr-data:
projects

/mnt/purr-data/projects:
projectA  projectB  projectC  projectD

/mnt/purr-data/projects/projectA:

/mnt/purr-data/projects/projectB:

/mnt/purr-data/projects/projectC:

/mnt/purr-data/projects/projectD:

/mnt/purr-fuse:
```

Check POSIX access as users
```
for i in rpwagner monawong pmeunier; do echo $i; sudo -u $i ls /mnt/purr-fuse/projects; echo; done
rpwagner
projectC  projectD

monawong
projectB  projectD

pmeunier
projectA  projectD
```

### Set hostname

`pgcs.stratus.jupyter-security.info`: `44.225.185.144`
```
44.225.185.144
sudo hostnamectl set-hostname pgcs.stratus.jupyter-security.info",
```


### Install GCS & Create Endpoint

Get a client ID and secret from [developers.globus.org](https://developers.globus.org)

Add repo
```
curl -LOs https://downloads.globus.org/globus-connect-server/stable/installers/repo/deb/globus-repo_latest_all.deb
sudo dpkg -i globus-repo_latest_all.deb
sudo apt-key add /usr/share/globus-repo/RPM-GPG-KEY-Globus
sudo apt update
sudo apt install globus-connect-server54
```

Endpoint setup

**Owner must be an OIDC/OAuth username or UUID from Globus**

Do not just use an email address

Pascal: `046f684d758c69a60175946503e700c9@ucsd.edu`
```
globus-connect-server endpoint setup "PURR GCS Test" \
    --organization "UCSD Research IT" \
    --client-id "89a5b977-7185-4c96-b9cd-8e655f4a81cb" \
    --owner 046f34a240f0615e01420b3ff4350922@ucsd.edu \
    --contact-email rick@ucsd.edu

sudo globus-connect-server node setup \
    --client-id "89a5b977-7185-4c96-b9cd-8e655f4a81cb" \
	-i 44.225.185.144
```

Login to localhost. This can be done as a non-privileged account on the system. That's better, since it will store tokens in your home directory.
```
globus-connect-server login localhost
```

Set as managed. Needed to enable Guest Collections. Can continue on until this is done. Can also be done in the web app. Can be done by subscription managers.
```
globus-connect-server endpoint set-subscription-id 6cb69fd5-a845-11e7-aeab-22000a92523b
Message: Updated Endpoint 89a5b977-7185-4c96-b9cd-8e655f4a81cb
```

### Storage gateway and mapped collection
Create POSIX storage gateway

- [Map UCSD email addresses to POISX usernames](gw-ucsd-email-id.json)
- [Base dir (`chroot`) of `/mnt/purr-fuse/projects`](gw-fuse-paths.json)

```
cat gw-fuse-paths.json 
{
  "DATA_TYPE": "path_restrictions#1.0.0",
  "none": [
    "/"
  ],
  "read_write": [
    "/mnt/purr-fuse/projects"
  ]
}

cat gw-ucsd-email-id.json 
{
  "DATA_TYPE": "expression_identity_mapping#1.0.0",
  "mappings": [
    {
      "source": "{email}",
      "match": "(.*)@ucsd\\.edu",
      "output": "{0}"
    }
  ]
}

globus-connect-server storage-gateway create posix \
   "PURR GCS FUSE Gateway" \
    --domain ucsd.edu \
    --posix-group-allow purr-users \
    --identity-mapping file:gw-ucsd-email-id.json \
   --restrict-paths file:gw-fuse-paths.json

Storage Gateway ID: b110ad11-e4c5-489c-8af3-b5de0d8a6895

globus-connect-server collection create \
   b110ad11-e4c5-489c-8af3-b5de0d8a6895 \
   /mnt/purr-fuse/projects \
  --allow-guest-collections \
  --enable-https \
   "PURR Test Mapped Fuse Collection"
Collection ID: 76062968-59fc-44aa-8633-3ca1e252be2b
```

- [PURR Test Mapped Fuse Collection](https://app.globus.org/file-manager?origin_id=76062968-59fc-44aa-8633-3ca1e252be2b)
- Collections at `https://app.globus.org/file-manager?origin_id={collection-id}`
- E.g, `https://app.globus.org/file-manager?origin_id=76062968-59fc-44aa-8633-3ca1e252be2b`

## Guest Collection using Service Account

```
- [ ] Create mapped collection
   - [ ] Base dir of `/mnt/purr-data`
   - [ ] Only accessible by `purr-data` users
- [ ] Create guest collection
   - [ ] Owned by `purr-data`'s Globus account (TBD)
   - [ ] Base dir of `/mnt/purr-data`

{
  "DATA_TYPE": "external_identity_mapping#1.0.0",
  "command": [
    "/usr/bin/python3",
    "/opt/globus/mapapp-example.py"
  ]
}


cd /opt/globus/
sudo wget https://docs.globus.org/globus-connect-server/v5.4/mapapp-example.py

globus-connect-server storage-gateway create posix \
   "PURR GCS POSIX Gateway" \
    --domain ucsd.edu \
    --user-allow purr-data \
    --identity-mapping file:gw-purr-data-id.json \
   --restrict-paths file:gw-posix-paths.json
Storage Gateway ID: cfdc425e-c1c5-470d-8af2-ff74929066ba

cat gridmap-cfdc425e-c1c5-470d-8af2-ff74929066ba 
046f34a240f0615e01420b3ff4350922@ucsd.edu purr-data

sudo cp gridmap-cfdc425e-c1c5-470d-8af2-ff74929066ba /etc/globus/

globus-connect-server collection create \
   cfdc425e-c1c5-470d-8af2-ff74929066ba  \
   /mnt/purr-data \
   --private \
   --enable-https \
   --allow-guest-collections \
   "PURR Test Service User Collection"

Collection ID: 9ba62f19-0df9-4e2c-ac18-801f728f9196


Guest Collection
PURR Projects Test Guest Collection
https://app.globus.org/file-manager/collections/b58f1e66-7b3e-48df-80e8-4e4d42406a87/overview
```

```
ubuntu@pgcs:~$ sudo mkdir /mnt/image-repository
ubuntu@pgcs:~$ sudo chown rpwagner:rpwagner /mnt/image-repository
ubuntu@pgcs:~$ cat image-repo-paths.json
{
  "DATA_TYPE": "path_restrictions#1.0.0",
  "none": [
    "/"
  ],
  "read_write": [
    "/mnt/image-repository"
  ]
}
```

```
ubuntu@pgcs:~$ cat external-map.json 
{
  "DATA_TYPE": "external_identity_mapping#1.0.0",
  "command": [
    "/usr/bin/python3",
    "/opt/globus/mapapp-example.py"
  ]
}
```


```
globus-connect-server storage-gateway create posix \
   "Image Repo POSIX Gateway" \
    --domain ucsd.edu \
    --user-allow rpwagner \
    --identity-mapping file:external-map.json \
   --restrict-paths file:image-repo-paths.json
Storage Gateway ID: 1267bbc5-226e-4354-bdb8-2b5cc16437a6
```

```
ubuntu@pgcs:~$ cat gridmap-1267bbc5-226e-4354-bdb8-2b5cc16437a6
"046f34a240f0615e01420b3ff4350922@ucsd.edu" rpwagner
ubuntu@pgcs:~$ sudo cp gridmap-1267bbc5-226e-4354-bdb8-2b5cc16437a6 /etc/globus
```

```
globus-connect-server collection create \
   1267bbc5-226e-4354-bdb8-2b5cc16437a6  \
   /mnt/image-repository \
   --private \
   --enable-https \
   --allow-guest-collections \
   "Image Repo Mapped Collection"
Collection ID: 4cc96c30-e1b5-451b-a5f0-8223acd0be14
```

```
ubuntu@pgcs:~$ sudo -u rpwagner  touch /mnt/image-repository/data.txt
```

```
$ globus session consent
'urn:globus:auth:scope:transfer.api.globus.org:all[*https://auth.globus.org/scopes/4cc96c30-e1b5-451b-a5f0-8223acd0be14/data_access]'

$ globus ls 4cc96c30-e1b5-451b-a5f0-8223acd0be14:
data.txt

```

```
Alternate Serverless Image Repository
https://app.globus.org/file-manager/collections/527fe9c0-5782-4a2a-a097-ea2f06fe68ab/overview
```


```
$ globus endpoint permission list 527fe9c0-5782-4a2a-a097-ea2f06fe68ab
Rule ID                              | Permissions | Shared With                                                        | Path        
------------------------------------ | ----------- | ------------------------------------------------------------------ | ------------
9eba1978-4446-11ed-89d4-ede5bae4f491 | r           | anonymous                                                          | /public/    
ada8f210-4446-11ed-89d4-ede5bae4f491 | r           | https://app.globus.org/groups/260da91f-3496-11ed-b941-972795fc9504 | /allusers/  
bbf8f6ee-4446-11ed-89d4-ede5bae4f491 | r           | https://app.globus.org/groups/cf9d1f5b-3496-11ed-b941-972795fc9504 | /restricted/
NULL                                 | rw          | 046f34a240f0615e01420b3ff4350922@ucsd.edu                          | /           
NULL                                 | rw          | 89a5b977-7185-4c96-b9cd-8e655f4a81cb@clients.auth.globus.org       | /           
(base) ITSC02C982DMD6M:~ rpwagner$ 
```


Previous
```
https://g-b0978f.0ed28.75bc.data.globus.org/serverless/
```

Alternate
```
https://g-079c7d.ca528.03c0.data.globus.org/public/BOUNDEDNESS285/beveil-anapaestical.png
```
