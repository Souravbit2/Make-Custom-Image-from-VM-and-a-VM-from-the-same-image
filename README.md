# Make-Custom-Image-from-VM-and-a-VM-from-the-same-image
Creating a custom image in Azure after setting up IIS, and then creating a VM from that image, is a common practice for standardizing deployments and ensuring consistent environments.1 This process typically involves generalizing the source VM to remove machine-specific identifiers.2
Here's a step-by-step guide:
**Part 1: Prepare the Source VM and Create the Custom Image**
	1. Create a Windows Server VM in Azure:
		○ Go to the Azure portal.
		○ Search for "Virtual machines" and select "Create" -> "Azure virtual machine."
		○ Fill in the basics:
			§ Resource group: Create a new one or choose an existing one (e.g., myIISImageRG).3
			§ Virtual machine name: (e.g., IISMasterVM)
			§ Region: Choose your desired region.
			§ Image: Select a Windows Server image (e.g., Windows Server 2022 Datacenter).
			§ Size: Choose a suitable VM size.
			§ Administrator account: Create a username and strong password.
			§ Inbound port rules: Allow RDP (3389) for remote access. You can also allow HTTP (80) now if you want to test IIS right away.
Review and create the VM.
<img width="1265" height="630" alt="image" src="https://github.com/user-attachments/assets/4b2fc579-a7b6-49d5-81df-c847efa648a4" />
<img width="1282" height="130" alt="image" src="https://github.com/user-attachments/assets/7f15ee48-e7bb-41f4-aadf-381593ac6c66" />
**2. Connect to the VM and Install IIS:**
		○ Once the VM is deployed, navigate to its overview page in the Azure portal.
		○ Click "Connect" -> "RDP" and download the RDP file.
		○ Open the RDP file and connect to your VM using the administrator credentials you set.
		○ Install IIS:
			§ Open Server Manager (it should launch automatically).
			§ Click "Add roles and features."
			§ Click "Next" until you reach the "Server Roles" section.
			§ Select "Web Server (IIS)."
			§ Add any required features.
			§ Proceed with the installation.
After installation, you can verify IIS by opening a web browser on the VM and navigating to localhost. 4You should see the default IIS welcome page.
<img width="777" height="603" alt="image" src="https://github.com/user-attachments/assets/788e1b26-bef5-494a-8db3-4b2ec9244c1f" />
**3. Customize IIS (Optional but Recommended):**
		○ This is where you make your custom configurations. For example:
			§ Deploy your web application to C:\inetpub\wwwroot.
			§ Configure application pools.
			§ Set up bindings.
			§ Install any necessary features, runtimes (e.g., .NET), or applications that your web server requires.
			§ Configure Windows Firewall rules if needed for your applications.
**4. Generalize the VM using Sysprep:**
		○ Important: Sysprep prepares the Windows installation to be duplicated. It removes unique system identifiers (like SIDs) so that new VMs created from this image are unique.5 After running Sysprep, the original VM will be unusable.
		○ On the VM, open an elevated Command Prompt or PowerShell.
		○ Navigate to C:\Windows\System32\Sysprep.6
		○ Run sysprep.exe.
		○ In the System Preparation Tool dialog box:
			§ Select Enter System Out-of-Box Experience (OOBE).
			§ Check the Generalize checkbox.
			§ From the Shutdown Options dropdown, select Shutdown.
			§ Click OK.
The VM will generalize, shut down, and deallocate.
<img width="759" height="268" alt="image" src="https://github.com/user-attachments/assets/9440bb96-5bf6-4763-b439-81b42b8e30da" />
**5. Capture the Image in Azure:**
		○ In the Azure portal, navigate to your VM (IISMasterVM in this example).
		○ The VM's status should now be "Stopped (deallocated)."
		○ Click the "Capture" button in the overview blade.
		○ On the "Create an image" blade:
			§ Name: Give your custom image a descriptive name (e.g., myCustomIISImage).
			§ Resource group: Choose the resource group where you want to store the image (it can be the same as the VM's or a dedicated one for images).7
Automatically delete this virtual machine after creating the image: Select this checkbox. Since the VM is generalized and unusable, you want Azure to delete it.
<img width="706" height="179" alt="image" src="https://github.com/user-attachments/assets/34b7581c-ab3d-4e3e-b4b5-733fbce7e0d1" />
			§ VM Name: This field might be pre-filled with the source VM name.
			§ Azure Compute Gallery (Shared Image Gallery):
				□ Highly Recommended for production and sharing: If you plan to share this image across subscriptions, regions, or use it for scaling, select "Yes, share it to a gallery as a VM image version."
				□ If "Yes":
					® Create new or select existing gallery: Create a new gallery (e.g., myIISGallery) or select an existing one.
					® Create new or select existing image definition: Create a new image definition (e.g., IISImageDefinition). This defines the type of image (OS type, publisher, offer, SKU).
					® Version number: Provide a version for your image (e.g., 1.0.0).8
				□ If "No" (to create a standalone managed image): This is simpler but less flexible for sharing or managing multiple versions.
		○ Click "Review + create," then "Create."
The image creation process will take some time. You'll see notifications in the portal when it's complete.
<img width="1150" height="323" alt="image" src="https://github.com/user-attachments/assets/73de8be5-430f-4dcd-888a-557a1b3834e3" />
**Part 2: Create a New VM from the Custom Image**	
1. Navigate to Images or Azure Compute Gallery:
		○ If you created a managed image (standalone): Search for "Images" in the Azure portal and select your custom image (myCustomIISImage).
		○ If you created an image in an Azure Compute Gallery: Search for "Azure Compute Galleries," navigate to your gallery, then your image definition, and then the image version.
	2. Create VM from Image:
		○ From the image's overview page, click "Create VM."
		○ This will open the "Create a virtual machine" blade, with the image pre-selected.9
		○ Basics tab:
			§ Resource group: Choose an existing or create a new resource group for your new VM (e.g., myNewIISVMsRG).
			§ Virtual machine name: (e.g., WebFrontend01)
			§ Region: Choose the region where you want to deploy the VM.
			§ Size: Select a VM size.
			§ Administrator account: Provide a new username and password for this new VM. This is crucial as Sysprep generalized the previous credentials.
			§ Inbound port rules: Ensure HTTP (80) is allowed. If you need RDP, also allow RDP (3389).
		○ Disks, Networking, Management, Advanced, Tags: Configure these tabs as needed. For networking, ensure the Virtual Network and Subnet are correctly selected, and the Network Security Group (NSG) allows inbound traffic on port 80.
		○ Review + create: Review your settings.
Create: Click "Create" to deploy the new VM from your custom IIS image.
<img width="1197" height="388" alt="image" src="https://github.com/user-attachments/assets/922824cb-8c84-4a7e-8c65-730b11d51ff6" />
	3. Verify the New VM:
		○ Once the new VM is deployed, get its public IP address from its overview page.
		○ Open a web browser and navigate to http://YOUR_NEW_VM_PUBLIC_IP.
You should see your custom IIS configuration or the default IIS page, confirming that IIS is installed and running as part of the image.
<img width="843" height="381" alt="image" src="https://github.com/user-attachments/assets/be1feed9-0075-4908-9cb1-365cdf7574f8" />








