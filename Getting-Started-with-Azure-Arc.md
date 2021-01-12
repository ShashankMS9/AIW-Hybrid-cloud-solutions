# Exercise 1: Connect On-Prem Servers to Azure with Arc

Azure Arc extends Azure Resource Manager capabilities to Linux and Windows servers, as well as Kubernetes clusters on any infrastructure across on-premises, multi-cloud, and edge. With Azure Arc, customers can also run Azure data services anywhere, realizing the benefits of cloud innovation, including always up-to-date data capabilities, deployment in seconds (rather than hours), and dynamic scalability on any infrastructure. Azure Arc for servers is currently in public preview.

## Task 1: Login and become familiar with Hyper-V Infrastructure

Hyper-V is Microsoft's hardware virtualization product. It lets you create and run a software version of a computer, called a virtual machine. Each virtual machine acts like a complete computer, running an operating system and programs. When you need computing resources, virtual machines give you more flexibility, help save time and money, and are a more efficient way to use hardware than just running one operating system on physical hardware. In this task, you will walk through on-prem environment which is hosted on Hyper-V. You will find three virtual machines hosted on Hyper-V server, which you will onboard to Azure Arc and play around.

1. You can see a virtual machine desktop 💻 (LabVM/ARCHOST) is loaded in the left side in your browser. Use this virtual machine throughout the workshop to perform the the lab. You can also connect to the virtual machine using RDP using the ARCHOST VM credentials provided on **Environment Details** tab. Double click on the **Hyper-V Manager** for VM desktop to start the Hyper-V Manager

    ![](media/cloudlab-vm-guide.png "Lab Environment")

1. Select **ARCHOST** to connect with Local Hyper-V server. In your Machine there could be a unique suffix added at end of **ARCHOST**, something like **ARCHOST-XXXXXX**.

    ![](media/archost-localserver.png "ARCHOST Server")

1. You will find two guest virtual machines running on the Hyper-V manager. Find list of guest virtual machines with private ip address.
     * **ubuntu-k8s** - ```192.168.0.8```
     * **winvm** - ```192.168.0.5```
     
    ![](media/guestvms.png "Guest VMs")
    
## Task 2: Onboard Linux Machine to Azure Arc

In this task, we will learn to onboard the Linux VMs and local kubernetes cluster to Azure Arc. So, here we will onboard **ubuntu-k8s** VM to Azure ARC.

1. From the start menu of the ARCHOST VM, search for **putty** and open it with double click or other way.

    ![](media/startputty.png "Search Putty")
     
1. In Putty Configuration tool, enter the **ubuntu-k8s** VM private IP - ```192.168.0.8```, make sure the Port value is ```22```. Once you entered the private IP of the ubuntuk8s vm, click on the Open to lunch the terminal.

    ![](media/putty-enter-ip.png "Enter ubuntu-k8s VM private IP")
    
1. Enter the **ubuntu-k8s** vm username - ```demouser``` in **login as** and then hit **Enter**. Now, enter the password - ```demo@pass123``` and press **Enter**. Remember password will be hidden and not be visible in terminal, don't worry about that.

    ![](media/enter-ubuntu-k8s-credentials.png "Enter ubuntu-k8s credentials")
    
    > Note: To paste any value in Putty terminal, just copy the values from anywhere and then right click on the terminal to paste the copied value.
    
1. Login to the **Root user account** using sudo command; enter following command and then provide **password** - ```demo@pass123``` when prompted for password.
     * Command: ```sudo su ```
     * Password: ```demo@Pass123```
    
    ![](media/root-login.png "Root Login")
    
1. There is file ```installArcAgentLinux.txt``` on ARCHOST VM desktop 💻. Open that and copy first 6 lines and paste in putty to declare the values of AppID, AppSecret, TenantID, SubscriptionID, ResourceGroup and location. You can also find the values of these variables in **Environment Details** tab and then use in next steps.

    ![](media/variables.png "variables")
    
1. Login to Azure using following command: ```az login -u $AppID --service-principal --tenant $TenantID -p $AppSecret```

    ![](media/azlogin.png "Login to Azure")
    
1. Now, downlaod the Azure Arc installation package for linux, run the below command:

    ```wget https://aka.ms/azcmagent -O ~/install_linux_azcmagent.sh```
    
    ![](media/download-arc-agent.png "Download ARC Linux Agent")
    
1. Install Azure Arc agent by running : ```bash ~/install_linux_azcmagent.sh```

    ![](media/run-installation.png "Install ARC Agent")
    
1. Once the installation is successful, you will see following message in terminal **Latest version of azcmagent is installed**.

    ![](media/arcagent-installed.png "ARC Agent latest version installed")    
    
1. Finally, connect the ubuntu-k8s machine to Azure Arc. Run following connect command.  Once you run the below command, it will take few minutes to onboard the machine to Azure Arc. 
    
    ```azcmagent connect --resource-group $ResourseGroup --tenant-id $TenantID --location $location --subscription-id $SubscriptionId -i $AppID -p $AppSecret```
      > Remember, we are using variables declared earlier in step 5. If you have connect with new putty session, you may have to run steps 4,5,6 again.
     
    ![](media/connected-azure-arc.png "Connected to Arc")

## Task 3: Onboard Kubernetes Cluster to Azure Arc
In this task, we will learn to onboard local kubernetes cluster to Azure Arc. So, here we onboard **MicroK8s** kubernetes cluster to Azure ARC which is hosted on **ubuntu-k8s** VM . For this task, Microk8s kubernetes cluster is already configured, also Arc enabled CLI extensions are installed.

 > Note: If you have closed the putty after completing **task 2**, then perform first 6 steps of task 2 again and then jump to this task. 

1. Now, we need to run the commands as demouser instead of root user, if you are logged in as root user, enter ```exit``` command to get back on **demouser**.

    ![](media/exit-from-root.png "Exit root user")

1. Update the Arc enabled Kubernetes CLI extensions, if there will be any update these commands will update the k8s extensions.
    ```az extension update --name connectedk8s
       az extension update --name k8sconfiguration
    ```
    ![](media/update-k8s-extensions.png "Update Az k8s extensions")
    
1. Now, check the status of Kubernetes cluster by running ```microk8s.status``` as demouser in **ubuntu-k8s** vm. You can proceed further if it is running, if it is in stopped state, you may have to run ```microk8s start``` command to run the kubernetes cluster.

    ![](media/k8s-status-running.png "check cluster cluster")
    
1. There is file ```installArcAgentLinux.txt``` on ARCHOST VM desktop 💻. Open that and copy first 7 lines and paste in putty to declare the values of AppID, AppSecret, TenantID, SubscriptionID, ResourceGroup and location and then **login to azure** using 7th line. You can also find the values of these variables in **Environment Details** tab and then use in next steps.

    ![](media/login-to-azure.png "Login to Azure")
    
1. Connect the Kubernetes cluster to Azure Arc by executing the following command, once you run teh command, it will take few minutes to onboard Kubernetes Cluster to onboard to Azure Arc.

      ```az connectedk8s connect --name microk8s-cluster --resource-group $ResourseGroup -l $location```
    
    ![](media/connect-k8s.png "Connect Kubernetes")
    
1. Once the previous command executed successfully, check the **provisioningstate** in output, it should be in succeeded status.

    ![](media/k8s-connected1.png "Kubernetes Cluster Connected")
    
You have successfully onboarded ubuntu-k8s vm and microk8s-cluster kubernetes cluster to Azure Arc.