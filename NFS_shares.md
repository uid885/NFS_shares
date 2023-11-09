#Author: Christo Deale <br>
#Date: 2023-11-09

# Setup NFS shares on RHEL 9 & configure Windows10/11 Clients to access
(Please note the share are open for rw access on network)

## Step 1: RHEL Server Side
Ensure the nfs-utils are installed
```
sudo dnf install nfs-utils rcpbind
```
You need to edit the _exports_ file listing the shares that needs to be used.
```
sudo vim /etc/exports
```
/shared_directory1 *(rw,all_squash,anonuid=65534,anongid=65534)<br>
/shared_directory2 *(rw,all_squash,anonuid=65534,anongid=65534)<br>
/shared_directory3 *(rw,all_squash,anonuid=65534,anongid=65534)<br>

After you specified the Directories that needs to be shared, you need to apply the changes by exporting the file system recursively. <br>
```
sudo exportfs -rv
```

After the export, you needs to ensure that the NFS server is started/enabled en verified that the service is running.
```
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
sudo systemctl status nfs-server rpcbind
sudo rpcinfo -p | grep nfs
```
Then you make sure the shared_directory is has the correct access rights.
```
sudo chmod -R o+w shared_directory1
```

To ensure that the NFS shares are accessable when your server's firewall is active, you need to make add the service:
```
sudo firewall-cmd --add-service=nfs --permanent
sudo firewall-cmd --add-service=rpc-bind --permanent
sudo firewall-cmd --add-service=mountd --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all | grep nfs
```
## Step 2: Windows Client Side
You need to disable the firewall. <br>
Navigate to the **Control Panel** > **Programs** > **Programs and Features** > **Turn Windows Features on/off** <br>
Select **Services for NFS** and click apply & okay. This will install the services needed.
![nfs](https://github.com/uid885/NFS_shares/assets/135722741/fc43fb43-3299-470b-9a8b-777222d8931e) <br>
I have found that sometimes its best to reboot the Windows box after the service was installed. <br>
Open **CMD** as Administrator. You should be able to see the listed shares on the server:
```
showmount -e server-ip
```

Now in order to actually access these theres thats a security edit you need to do one time to enable windows to do map the share & access. <br>
```
secedit /configure /cfg %windir%\inf\defltbase.inf /db defltbase.sdb /verbose
```
**Reboot the windows box again, i know right**

Start > run > \\\server-ip <enter>  & press _enter_ <br>
Now you can **right-click** & share.




