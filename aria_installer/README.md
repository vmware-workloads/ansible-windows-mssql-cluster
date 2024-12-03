AAP vRO Workflow Installer
==========================

Installs the given aria blueprint and vro workflows into aria automation.  
The automation will then be available in Service Broker.

## Prerequisites

-   AAP installed and running

-   Aria / VCF Automation configured with VM flavour mappings: small/medium/large

-   Note: pulls Ubuntu images from the web. If required, alter the template to
    use a local image (image mapping in VCF Automation required)

-   Project added to ansible, credential added with username/password and a
    template name of 'DataStax Enterprise Template'

-   NSX Networking available and project setup to be able to create new routable
    segments

-   Python 3.10 or higher on the machine where the script will be executed

-   Aria Automation 8.16 or higher
  
-   Appropriate Cloud Zone added to Aria Project

-   vRO connected to vCenter (as per
    https://docs.vmware.com/en/VMware-Aria-Automation/8.18/Using-Automation-Orchestrator-Plugins/GUID-C2EC619C-EAB0-43BB-98E4-7E54C6AA4CFD.html)

Files
-----

-   `config.json`: installer configuration file.

-   `vro_aria.py`: aria installer.

-   `aria_blueprint.yaml`: aria blueprint to be installed.

-   `vro_workflow.json`: aria vro workflow to be installed.

-   `vro_delete_workflow.json`: aria delete workflow to be installed.

Deployment
----------

Open the file 'config.json', and enter the appropriate values for the following
parameters:

`aria_base_url`: Enter the base url of your Aria deployment  
`aria_username`: Enter the username of your Aria deployment  
`aria_password`: Enter the password of your Aria deployment

  The other config.json parameters are optional and can be left as is or
modified as per your requirements.

After updating the configuration file, run the installer script:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ bash
python3 vro_aria.py
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
