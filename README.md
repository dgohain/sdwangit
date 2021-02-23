# SDWAN Day0 Templates and Policies

# Objective 

* Import Branch and DC Day 0 Templates which can be used for configuring Branch and DC routers for below requirements. 

    - On Branch and DC routers, Configure 2 WAN Interfaces (TLOCs) with color Public Internet and Biz Internet.  
    - On Branch Routers, Configure  Internal Service VPN 10 and Guest Service VPN 20.
    - On Branch Routers, Enable Direct Internet Access(DIA) for users in Guest Service VPN 20. 
    - On DC Routers, Configure Internal Service VPN 10 and Advertise Default Route in Service VPN 10 to Branches.

* Import Centralized Control Policy for Hub-n-Spoke Topology with DC1 preferred over DC2.

### Day 0 - Base Configuration Topology

![](images/Topology.png)

# Requirements

To use this code you will need:

* Python 3.7+
* vManage version: 20.4 or above
* vManage user login details. (User should have privilege level to Configure Templates and Policies)

# Install and Setup

- Clone the code to local machine.

```
git clone https://wwwin-github.cisco.com/msuchand/SDWAN-Day0-Templates.git
cd SDWAN-Day0-Templates
```
- Setup Python Virtual Environment (requires Python 3.7+)

On Linux and macOS environments:

```
python3.7 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
```

On Windows environment:

```
py -3.7 -m venv venv
.\venv\Scripts\activate
pip3 install -r requirements.txt
```

- Use below env commands to provide the login details of vManage.

## Example:

On Linux and macOS environments:

```
export VMANAGE_HOST=<vmanage IP address/DNS>
export VMANAGE_PORT=<vmanage Port and default value is 443>
export VMANAGE_USERNAME=<vmanage-username>
export VMANAGE_PASSWORD=<vmanage-password>
```

On Windows environment:

```
set VMANAGE_HOST=<vmanage IP address/DNS>
set VMANAGE_PORT=<vmanage Port and default value is 443>
set VMANAGE_USERNAME=<vmanage-username>
set VMANAGE_PASSWORD=<vmanage-password>
```

# Import DC Day 0 Templates

In the python virtual env, run the command `vmanage import templates -f DC-Day0-Template.json` to import DC Routers Day 0 Templates.

```
(venv)$ vmanage import templates -f DC-Day0-Template.json
Importing templates from DC-Day0-Template.json
Feature Template Updates: 7
Device Template Updates: 2
```

# Import Branch Day 0 Templates

In the python virtual env, run the command `vmanage import templates -f Branch-Day0-Template.json` to import Branch Routers Day 0 Templates. 

```
(venv)$ vmanage import templates -f Branch-Day0-Template.json
Importing templates from Branch-Day0-Template.json
Feature Template Updates: 9
Device Template Updates: 13
```

After running the above commands, below Feature templates and Device templates will be imported on vManage.

## Feature Templates

![](images/feature-templates.png)

## Device Templates

![](images/device-templates.png)

In order to use the Branch and DC Day 0 Templates, below are the variable values which needs to be provisioned when attaching devices to these templates. 

![](images/variables.png)

# Import Centralized Control Policy

- Create **config_details.yaml** using below sample format to provide inputs for creating Centralized Control Policy for Hub-n-Spoke Topology with DC1 preferred over DC2.

## Example:

```
#DC1 pref over DC2  policy

DC1_site_id: 100
DC2_site_id: 200
Spokes_site_id: 400-500
```

In the python virtual env, run the python script create-policies.py using `python3 create-policies.py` to import Hub-n-Spoke Policy with DC1 Preferred over DC2. 

```
(venv) python3 create-policies.py
Policy List Updates: 3
Policy Definition Updates: 3
Central Policy Updates: 1
Local Policy Updates: 0
```

Below is the sample Control Policy created by the script based on provided DC and Branch Site ID values in .yaml file.

```
policy
    control-policy DC1-Pref-over-DC2-Policy
        sequence 1
        match route
        site-list DC1-Site-list
        prefix-list _AnyIpv4PrefixList
        !
        action accept
        set
        preference 100
        !
        !
        !
        sequence 11
        match route
        site-list DC2-Site-list
        prefix-list _AnyIpv4PrefixList
        !
        action accept
        set
        preference 75
        !
        !
        !
        sequence 21
        match tloc
        site-list DC1-Site-list
        !
        action accept
        !
        !
        sequence 31
        match tloc
        site-list DC2-Site-list
        !
        action accept
        !
        !
    default-action reject
    !
    control-policy Block-DC2-to-DC1-Tunnels
        sequence 1
        match tloc
        site-list DC1-Site-list
        !
        action reject
        !
        !
    default-action accept
    !
    control-policy Block-DC1-to-DC2-Tunnels
        sequence 1
        match tloc
        site-list DC2-Site-list
        !
        action reject
        !
        !
    default-action accept
    !
    lists
    site-list DC1-Site-list
    site-id 100 
    !
    site-list DC2-Site-list
    site-id 200 
    !
    site-list Spokes-list
    site-id 400-500 
    !
    prefix-list _AnyIpv4PrefixList
    ip-prefix 0.0.0.0/0 le 32 
    !
    !
    !
    apply-policy
    site-list Spokes-list
    control-policy DC1-Pref-over-DC2-Policy out
    !
    site-list DC2-Site-list
    control-policy Block-DC2-to-DC1-Tunnels out
    !
    site-list DC1-Site-list
    control-policy Block-DC1-to-DC2-Tunnels out
    !
    !
```

# References

Above commands used to import templates and policies are based on Python Viptela SDK and for more details on Viptela SDK please refer https://github.com/CiscoDevNet/python-viptela 
