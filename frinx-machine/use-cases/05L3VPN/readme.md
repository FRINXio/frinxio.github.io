## Demo “L3VPN”

<!-- Before running L3VPN, run **Allocate_Root_Pool / 1** workflow and check if lab-vmx1 (device) is already installed.

![FRINX Machine dashboard](../demo_pics/allocate_root_pool_workflow.png)
-->

- On the FRINX Dashboard, open menu in the top-left corner and select on **L3VPN Automation**.
- Select **Services**.
- Select **+ Add service**.
- Fill in the information as shown below. Select the chain icon to automatically generate the **VPN ID**.

![FRINX Machine dashboard](../demo_pics/create_vpn_service.png)

- Select **Save changes**.
- You are redirected to the previous page.

![FRINX Machine dashboard](../demo_pics/vpn_services.png)

- Select **Commit changes**.
- Select **Commit changes** again.
- After committing, you can see all executed tasks and sub-workflows. Select **Go to detail** to review individual processes.

![FRINX Machine dashboard](../demo_pics/l3vpn_status.png)

### Step 1.

- Navigate back to the **L3VPN Automation** page.
- Select **Sites**.
- Locate the **test_site_3b9UQL4i** entry.

![FRINX Machine dashboard](../demo_pics/sites.png)

- For **test_site_3b9UQL4i**, select **Manage** and **Site network access**.

![FRINX Machine dashboard](../demo_pics/sites_network_accesses.png)

- Select **Add network access**.

### Step 2.

Enter the following settings:

**General and Service**

**VPN Attachment:** GNS00001002

**BTM Circuit Reference:** CES00000000-05

**Devices:** Select one of the CPE devices.

**SVC Input Bandwith (Mbsp):** 1000

**SVC Output Bandwith (Mbps):** 1000

![FRINX Machine dashboard](../demo_pics/create_site_network_access.png)

**Routing Protocol:**

- Select **+ Create Static Protocol**.

**Static Routing LAN:** 10.0.0.0/8

**Static Routing Next Hop:** 10.0.0.1

**Static Routing Lan Tag:** 999

**Bgp Profiles:** 300ms

**Maximum Routes:** 2000

![FRINX Machine dashboard](../demo_pics/routing_protocol.png)

**IP Connection**

To automatically generate a provider and customer address, select the chain icon:

![FRINX Machine dashboard](../demo_pics/ico_chain.png)

**BFD Profile:** 500ms

![FRINX Machine dashboard](../demo_pics/bgb_profile.png)

Select **Save Changes**.

### Step 3.

Select **Commit Changes**.

![FRINX Machine dashboard](../demo_pics/sites_network_accesses_2.png)

Wait until all tasks are completed.

![FRINX Machine dashboard](../demo_pics/status_l3vpn.png)
