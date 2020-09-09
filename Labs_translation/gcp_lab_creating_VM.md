# LAB: Creating Virtual Machines 

## Objectives

In this lab, you explore the available options for VMs and see the differences between locations.

In this lab, you learn how to perform the following tasks:

    - Create several standard VMs

    - Create advanced VMs



## Task 1. Create a utility virtual machine

- Create a VM

	*gcloud compute instances create my-vm-0 --zone=us-central1-c --machine-type=n1-standard-1 --subnet=default --no-address* 

- Explore VM

	*gcloud compute instances describe my-vm-0*

## Task 2: Create a Windows virtual machine

- Create a VM

	*gcloud compute instances create my-vm-1 --zone=europe-west2-a --machine-type=n1-standard-1 --subnet=default --network-tier=PREMIUM --tags=http-server,https-server --image=windows-server-2016-dc-core-v20200813 --image-project=windows-cloud --boot-disk-size=100GB --boot-disk-type=pd-ssd* 

	*gcloud compute firewall-rules create allow-http --direction=INGRESS --action=ALLOW --rules=tcp:80 --target-tags=http-server*

	*gcloud compute firewall-rules create allow-https --direction=INGRESS --action=ALLOW --rules=tcp:443 --target-tags=https-server*

- Set the password for the VM 

	*gcloud compute reset-windows-password my-vm-1*


## Task 3: Create a custom virtual machine

- Create a VM
	
	*gcloud compute instances create my-vm-2 --zone=us-west1-b --machine-type=custom-6-32768 --subnet=default* 

- Connect via SSH to your custom VM

	*gcloud compute ssh my-vm-2 --zone=us-west1-b*


- To see information about unused and used memory and swap space on your custom VM, run the following command:

  	*free*

- To see details about the RAM installed on your VM, run the following command:

	*sudo dmidecode -t 17*

- To verify the number of processors, run the following command:

	*nproc*

- To see details about the CPUs installed on your VM, run the following command:

	*lscpu*

- To exit the SSH terminal, run the following command:

	*exit*





