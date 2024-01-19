# VirtualBox-based Active Directory Lab

## Overview  

To replicate a genuine Active Directory environment, I engaged in this lab to guide through the fundamental steps of establishing and configuring a network based on Windows. The goal is twofold: firstly, to offer a detailed insight into the workings of Active Directory, and secondly, to delve into the broader realms of Windows networking. Utilizing Oracle's VirtualBox, I simulated an environment consisting of two virtual machines. The first machine operates on Windows Server 2019, serving as the Domain Controller, while the second runs Windows 10 to simulate a client experience.

## Objective

In the course of this lab, we will undertake the following tasks:

- Establish Active Directory Domain Services from the ground up.
- Generate a dedicated Domain Admin account to enhance security and control measures.
- Implement a DHCP server to dynamically allocate IP addresses.
- Illustrate the creation of an Organizational Unit for improved user management.
- Explore the complexities of Routing and Remote Access to replicate a corporate intranet environment.
- Configure the Network Interface Card (NIC) to ensure both local and internet connectivity.
- Showcase mass user management by utilizing PowerShell to batch-add over 1000 users.
- Engage with our client machine, simulating a login process using one of the newly added users.

## Technologies Employed

- Oracle VirtualBox
- Windows Server 2019
- Windows 10
- PowerShell

## Installation

### Virtual Machine Setup in VirtualBox
- Open Oracle VirtualBox and click "New."
- Name the first machine as "DC," select Microsoft Windows as the type, and choose Windows (64-bit) as the version.
- Allocate a minimum of 2 GB of RAM and create a new virtual hard disk with a capacity of at least 50 GB.
- Repeat the process for the Windows 10 machine, naming it "CLIENT1."

### Operating System Installation and Initial Setup

I began the installation process by associating the ISO files with their corresponding Virtual Machines. Notably, setting up Windows Server 2019 demanded extra care to guarantee appropriate administrative access. Here are the details:

- **Windows Server 2019 Install:** Proceed through the on-screen instructions until you reach the account setup stage.

(Insert Pic)

- **Establish an Administrative Account:** As a component of the installation process, you will receive a prompt to generate an administrator account. At this point, I supplied a username along with a strong and distinct password to bolster security. Given that this account possesses elevated permissions, it is imperative to ensure its thorough protection.

(Insert Pic)
(Insert Pic)

### Setting Up Operating Systems and Configuring Network Interface Cards (NICs)

I commenced the installation process by associating the ISO files with their respective Virtual Machines (VMs). The on-screen instructions were generally straightforward, resulting in the successful installation of both operating systems.

Furthermore, for our Domain Controller, it is crucial to configure dual Network Interface Cards (NICs) to ensure effective network management. Here's the configuration I implemented:

- **NIC for Open Internet:** The first NIC is set up to utilize Network Address Translation (NAT), allowing our Domain Controller to connect to the open Internet.

- **NIC for Internal Network:** The second NIC is exclusively dedicated to our internal virtual network, ensuring isolated communication within our simulated environment.

This dual NIC setup provides the Domain Controller with the network versatility necessary for real-world applications, aligning seamlessly with the objectives of our lab.

This configuration has established a secure yet flexible network topology, which is indispensable for our Active Directory Lab.

## Incorporating Active Directory Domain Services

Upon initiating Windows Server, I navigated to the Server Manager, where I carefully included the "Active Directory Domain Services" role.
(Insert Pic)
(Insert Pic)
(Insert Pic)
(Insert Pic)

## Creating a Domain Admin Account
- Launch Active Directory Users and Computers.
- Right-click and choose New > User.
- Input the necessary details for your new Domain Admin account and establish a robust password.
- Right-click on the new account and click Properties then left-click on Member Of.
- In the empty box under the prompt “Enter the object names..” enter Domain Admins, and click OK to apply changes.

(Insert Pic)
(Insert Pic)
(Insert Pic)

## Configure DHCP Server
Returning to the Server Manager, I implemented the "DHCP Server" role to guarantee dynamic allocation of IP addresses for our client machine.

(Insert Pic)
(Insert Pic)

## Organizing the Organizational Unit
- Open Active Directory Users and Computers.
- Right-click on your domain, navigate to New > Organizational Unit, and establish a new OU.

## Setting up Routing and Remote Access
- Open Server Manager and include the "Routing" role.
- Configure Routing and Remote Access to establish an internal network.

(Insert Pic)
(Insert Pic)
(Insert Pic)

## Setting up NIC for Internet Access
- Open Network and Sharing Center and navigate to "Change adapter settings."
- Adjust the NIC settings to establish a connection to the external internet.

## Adding Users using PowerShell
In organizational settings, managing hundreds or even thousands of users manually can be inefficient. To streamline this process, I've integrated a PowerShell script for batch user creation. Let's analyze the script [AddUsers.ps1](https://github.com/sindycp/Active-Directory-Lab/blob/main/AddUsers.ps1) to comprehend its structure and functionality:

### Defining Variables:
```powershell
# ----- Customize these Variables for Your Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #
```
In this section, two variables are being established:

`$PASSWORD_FOR_USERS = "Password1"`: This variable assigns a default password ("Password1") to all users. While using a universal password is not recommended for production, it is suitable for this lab.

`$USER_FIRST_LAST_LIST = Get-Content .\names.txt`: This line reads a file named [names.txt](https://github.com/sindycp/Active-Directory-Lab/blob/main/names.txt), which contains a list of first and last names. Each entry in this file is expected to be formatted as "FirstName LastName", with one pair per line.

### Transform Plain Text Password:
```powershell
$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
```

Due to security considerations, PowerShell commands associated with user management frequently necessitate passwords in the form of a secure string rather than plain text. In this instance, we are converting the plain text password from the `$PASSWORD_FOR_USERS` variable into a secure string and storing it in the `$password` variable.

### Create Organizational Unit (OU):
```powershell
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false
```
In this command, an Organizational Unit (OU) named `_USERS` is being established. The `-ProtectedFromAccidentalDeletion $false` parameter ensures that the OU is not safeguarded from accidental deletions. While useful for a lab environment, in a production scenario, consider enabling protection.

### Loop Through Names and Create Users:
```powershell
foreach ($n in $USER_FIRST_LAST_LIST) {
    # Actions to be performed...
}
```
For each name in our list `($USER_FIRST_LAST_LIST)`, the subsequent actions are executed.

### Extract the First and Last Names:
```powershell
$first = $n.Split(" ")[0].ToLower()
$last = $n.Split(" ")[1].ToLower()
```
The `.Split(" ")` method divides the full name into a first and last name based on the space. The `.ToLower()` method then converts these names to lowercase.

### Construct the Username:
```powershell
$username = "$($first.Substring(0,1))$($last)".ToLower()
```

This line forms a username by taking the first letter of the first name and appending the full last name, all in lowercase. For instance, for the name `"John Doe"` the username would be `"jdoe"`.

### Display Progress in the Console:
```powershell
Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
```

This line exhibits a message in the console to indicate which user is currently being created.

(Insert Pic)

### Generate the New User:
```powershell
New-AdUser ...
```

The `New-AdUser` cmdlet is employed to generate a new user in Active Directory. The script defines multiple parameters, including given name, surname, display name, etc., and situates the user within the previously established `_USERS` OU. The `-PasswordNeverExpires $true ` parameter is utilized to prevent the expiration of the user's password, suitable for a lab environment. However, in a live production setting, it is advisable to set a password expiration.

### What Is the Purpose of This Action?
The primary objective of this script is to showcase the automation capabilities in Active Directory management, particularly when handling a substantial number of users. Instead of engaging in the tedious and error-prone process of manually entering each user, this script simplifies and automates the task, guaranteeing uniformity and efficiency. It highlights the effectiveness of PowerShell scripting in administering Windows environments and provides a tangible example of its practical application in real-world scenarios.

### Interaction with the Client and User Experience
After preparing our Windows 10 VM, I set it up to establish a connection with our server's environment. Following that, I logged off and logged back in, replicating the experience of one of our users created in batches.

(Insert Pic)

## Final Reflections and Conclusions

### Attained Goals
Throughout this lab, we navigated the fundamentals of establishing and configuring a Windows-based Active Directory environment. The step-by-step walkthrough covered everything from initial setup and domain configurations to intricate networking adjustments and automation. The use of the PowerShell script for bulk user creation showcases the scalability and automatability of an Active Directory environment.

### Educational Insights
The lab provided a valuable hands-on experience, demystifying the often intricate domain of Windows networking and Active Directory services. It offered insights into foundational aspects such as DHCP setup, user management, Organizational Unit (OU) creation, and networking intricacies. Most notably, it underscored how automation can significantly enhance administrative efficiency, a crucial lesson for IT professionals.

### Practical Relevance
The skills and knowledge acquired in this lab extend beyond the experimental realm. These competencies are highly applicable in real-world scenarios, where Active Directory plays a pivotal role in organizational IT infrastructures. The focus on automation, in particular, empowers individuals to effectively manage large-scale environments.

### Future Exploration
While this lab covers substantial ground, it's essential to recognize that Active Directory and Windows networking encompass even more advanced topics. Exploring areas such as Group Policies, advanced security measures, multi-site configurations, and integration with cloud services offers avenues for continued learning.

### Closing Statements
In conclusion, this lab serves as a foundational exploration into Windows-based networking and Active Directory. It not only imparts technical expertise but also fosters a strategic understanding of the roles these services play within broader IT ecosystems.

While Active Directory's complexity may appear overwhelming, as evidenced in this lab, adopting a structured and hands-on approach can greatly aid in mastering it. For individuals entering the IT field or looking to improve their skills, gaining a profound understanding of Active Directory is akin to a rite of passage, and this lab aims to facilitate that transition with ease.


