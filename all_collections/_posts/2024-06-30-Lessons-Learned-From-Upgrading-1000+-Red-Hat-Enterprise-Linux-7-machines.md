Lessons learned from upgrading 1000+ Red Hat Enterprise Linux machines

1. Once you go through 7 - 8 upgrade then 8 - 9 would be easier 
2. RedHat states RHEL7-8 cannot be automated, imo 90% of use cases can be automated but it might differ depending on the inventory
3. Typical automation time that take for all the scripts to upgrade takes around 30 to 35 minutes

Process of the upgrade (general, without environment specific):
1. Pre upgrade preparationmigrate nicsyum updateinstall leappunmount obsolete kernel drivers
2. Leapp pre upgrade
3. Leapp upgrade
4. RebootCrucial step where actual upgrade is being performed (installation and scripts) using dedicated runtime environment
5. Post upgrade cleanupremove RHEL7 packagesremove leap

Typical errors, RedHat "stack overflow" is your friend:
1. Human errors, weird configuration issues
2. NFS volumes are mounted, automating that might be tricky
3. Newest kernel might be available through yum and installed automatically but grup does not defaults it by itself - need manual intervention
4. Some problems during upgrade reboot might occur - even leap pre upgrade and leap upgrade returns no errors - most probably due to conflicting packages, os boots into single user mode
5. 7 and 8 uses different NIC naming convention, but this migration can be automated with pointers from RH "SO"
6. RHSM might occasionally fail (500 HTTP error)
7. RHSM might offer newer 8.x release, i.e. if your upgrade automation scripts were written for upgrading into 8.9 and RHEL 8.10 is released, then the 8.10 gets picked up and those scripts might need fixes
