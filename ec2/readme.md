###### Instances
![image](https://github.com/user-attachments/assets/f292dd88-2c7e-45f1-b0d4-36e7636b8040)

###### Launch a new instance
![image](https://github.com/user-attachments/assets/c566265b-fb55-4b61-8940-4018834af097)
![image](https://github.com/user-attachments/assets/d9681604-d595-49b7-b18d-128269b71672)
![image](https://github.com/user-attachments/assets/1fce0d7b-eaa4-4906-a9a4-4426cca83e0c)
![image](https://github.com/user-attachments/assets/994effce-e0b8-43ba-afc8-1b6ed596d0a6)
![image](https://github.com/user-attachments/assets/90b37164-273d-4651-b9be-7ee2a5e57885)


> [!WARNING]
> In the picture below the drop-down list named "Capacity reservation" show nothing but in a production environment is advisable to reserve capacity or oterwise the expenditure in on-demand instances will become the solution more pricey than expected. Doing capacity reservation the cost of implementation can be cut down to even a half in some scenarios. Everything depends on if the reservation is completely paid in advance, partially paid or it's only reservation with no upfront payment. Please review the FinOps aspect of the solution. 

![image](https://github.com/user-attachments/assets/b9533e0b-a052-48e9-83dd-637ce3147837)

###### User Data script for instance launch
```
#cloud-config
package_update: true
package_upgrade: true
runcmd:
- file_system_id_1=fs-071a426xxxxxxxxxx
- efs_mount_point_1=/opt/efs
- mkdir -p "${efs_mount_point_1}"
- apt -y install nfs-common
- test -f "/sbin/mount.efs" && printf "\n${file_system_id_1}:/ ${efs_mount_point_1} efs tls,_netdev\n" >> /etc/fstab || printf "\n${file_system_id_1}.efs.eu-south-2.amazonaws.com:/ ${efs_mount_point_1} nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport,_netdev 0 0\n" >> /etc/fstab
- test -f "/sbin/mount.efs" && grep -ozP 'client-info]\nsource' '/etc/amazon/efs/efs-utils.conf'; if [[ $? == 1 ]]; then printf "\n[client-info]\nsource=liw\n" >> /etc/amazon/efs/efs-utils.conf; fi;
- retryCnt=15; waitTime=30; while true; do mount -a -t efs,nfs4 defaults; if [ $? = 0 ] || [ $retryCnt -lt 1 ]; then echo File system mounted successfully; break; fi; echo File system not available, retrying to mount.; ((retryCnt--)); sleep $waitTime; done;
- apt -y install wget gpg coreutils htop mc net-tools tree dnsmasq jq
- wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
- echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list
- apt -y update && apt -y upgrade && apt -y install nomad consul vault libffi-dev libssl-dev python3-dev python3 python3-pip python3-paramiko docker-compose-v2 docker.io docker-buildx docker-clean libnfs-utils nfs-common
``` 
