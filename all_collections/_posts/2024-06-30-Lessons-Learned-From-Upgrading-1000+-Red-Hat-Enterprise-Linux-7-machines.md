*General notes*

1. This blog post covers RHEL7 to RHEL8 in-place upgrade automation process, but lessons learned can be reused for RHEL8 to RHEL9 and future upgrade iterations. 
2. RedHat states RHEL7-8 cannot be automated, imo 90% of use cases can be automated but it might differ depending on the machine environment.
3. Typical automation time that take for all the scripts to upgrade takes around 30 to 35 minutes.

*Process of the upgrade (general, without environment specific):*

1. Register into Red Hat Subscription Manager (if the machine was not previously registered). This step ensures all the required remote packages being available for the installation.
2. Pre upgrade preparation:
    * Rename NICs to match RHEL8 naming convention.
    * Update all the RHEL7 system packages into the newest, whatever available 7.x via **yum update**.
    * install leapp package (which is the crucial application that handles all the system packages ugprade).
    * unmount obsolete kernel drivers (floppy, pata_acpi).
    * provide leapp answers into answerfile.
2. Execute *leapp pre upgrade* which handles the system sanity check for the upgrade. If the system is not eligible for the upgrade (i.e. due to package incompatibility issues) then this step will fail and further, custom pre upgrade steps would need to be perfomed.
3. Execute *leapp upgrade* which will prepare the system for the upgrade that will happen in the next step.
4. Execute *reboot*, which is a crucial step where the actual upgrade is being performed. Here your machine will be booted into dedicated runtime upgrade environment where all the previously downloaded packages would be installed and further system configuration would be performed. This step turns the SSH off and might even take 20 minutes, so keep it in mind while you're machine gives you impression that is not waking up for a longer time. Direct access to the machine console (or through your cloud provider web console) will reveal all the details about the upgrade process.
5. Post upgrade cleanup:
    * Remove remaining RHEL7 packages
    * Remove old RHEL7 kernels
    * remove leapp package, which is obviously not needed anymore once the machine was upgraded into RHEL8.

*Typical errors*

1. One of the most common and worst type of problems with your upgrade are human admin errors. I.e. you might experience corrupted system configuration files (such as invalid entries, wrong syntax) which sometimes make debuigging of the upgrade process harder and specific to any human error you're dealing with. This obviously can't be automated.
2. There are NFS volumes are mounted, automating that might be tricky
3. Leapp might complain about newest kernel being installed but not being selected by default in Grub. Leapp requires the machine for upgrade process to have the latest installed kernel running and that might be problematic, because once you execute *yum update* in the pre upgrade process the newest kernel will be installed, but the Grub configuration will stay intact, which will result in the older kernel running. In that case you'll either have to update the Grub configuration manually.
4.During upgrade reboot phase some problems might occur - even if *leapp preupgrade* and *leapp upgrade* command execution returns clean. This happens usually due to conflicting packages and make the upgrade process stuck at single user mode without SSH access. In conclusion, if you're machine is rebooting after the *leapp upgrade* for more than 25 minutes then it's worth checking the machine console output (most probably you'll be using you cloud provider web console access) and seeing if the upgrade process went into single user root mode. If that's the case then you'll be able to recover from that situation by executing *reboot* again to see if machine is booting into RHEL7 or RHEL8. If the machine booted into RHEL8 then you're fine, but if you're machine is still at RHEL7 then you'll have to check */var/log/leapp/leapp-upgrade.log* and &/var/log/leapp-report.txt* files to determine what caused an error, then fix it manually and rerun the whole upgrade process by executing *leapp upgrade*.
5. RHEL7 and RHEL8 uses different NIC naming convention, but this migration can be automated using some tips provided in Red Hat Customer Portal.
6. RHSM might occasionally fail (500 HTTP error)
7. RHSM might update their 8.x release, i.e. if your upgrade automation scripts were written for upgrading into 8.9 and RHEL 8.10 will be released, then the 8.10 gets picked up automatically and your scripts that were written having 8.9 in mind might need some fixing.

*Tips*

1. *Red Hat Customer Portal* is your friend: most of the common problems that can or can't be automated were already solved and shared there. 
2. */var/log/leapp/leapp-preupgrade.log*, */var/log/leapp/leapp-upgrade.log* and */var/log/leapp/leapp-report.txt* are *invaluable* sources of hints why you're single upgrade steps are failiing. Check them thoroughly any time you don't understand the errors thrown by leapp into the console - those logfiles often contain stack traces and more detailed information on why leapp is failing.