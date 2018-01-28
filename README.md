# Spectre and Meltdown security patch management

This repository uses Ansible playbooks to check and enable or disable patches
for CVE-2017-5754 CVE-2017-5715 and CVE-2017-5753 in Red Hat Linux.

Red Hat has created updated kernels available to address these security
vulnerabilities. These patches are enabled by default, to provide the highest
security possible.

The (kernel and microcode) updates may negatively impact certain performance
characteristics when compared to unpatched vunernable systems. Therefore some
customers may elect to disable these patches when they are confident that their
systems are protected by other means.

## Page Table Isolation (pti)

The pti_enabled controls the Kernel Page Table Isolation feature, which
isolates kernel pagetables when running in user space. This feature addresses
CVE-2017-5754, also called variant #3, or **Meltdown**.

Customers and vendors can disable the PTI feature by passing “nopti” to the
kernel command line at boot, or dynamically with the runtime debugfs control
below:

```
# echo 0 > /sys/kernel/debug/x86/pti_enabled
```

## Indirect Branch Restricted Speculation (ibrs)

This protects uservspace from hyperthreading/simultaneous multi-threading
attacks as well, and is also the default on AMD processors (family 10h, 12h
and 16h). This feature addresses CVE-2017-5715, variant #2.

Customer and vendors can disable the ibrs implementation in microcode by
passing "noibrs" to the kernel command line at boot, or dynamically with the
debugfs control below:

```
# echo 0 > /sys/kernel/debug/x86/ibrs_enabled
```

## Indirect Branch Prediction Barriers (ibpb)

When ibpb_enabled is set to 1, an IBPB barrier that flushes the contents of the
indirect branch prediction is run across user mode or guest mode context
switches to prevent user and guest mode from attacking other applications or
virtual machines on the same host. In order to protect virtual machines from
other virtual machines, ibpb_enabled=1 is needed even if ibrs_enabled is set
to 2.

If ibpb_enabled is set to 2, indirect branch prediction barriers are used
instead of IBRS at all kernel and hypervisor entry points (in fact, this
setting also forces ibrs_enabled to 0). ibpb_enabled=2 is the default on CPUs
that don’t have the SPEC_CTRL feature but only IBPB_SUPPORT. ibpb_enabled=2
does not protect the kernel against attacks based on simultaneous multi-threading
(SMT, also known as hyperthreading); therefore, ibpb_enabled=2 provides less
complete protection unless SMT is also disabled. This feature addresses
CVE-2017-5715, variant #2.

Customer and vendors can disable the ibpb implementation in microcode by
passing "noibpb" to the kernel command line at boot, or dynamically with the
debugfs control below:

```
# echo 0 > /sys/kernel/debug/x86/ibpb_enabled
```

### check_spectre_meltdown_status.yml
Use this playbook to check the flags for the 3 system settings.
```
ansible-playbook -i hosts check_spectre_meltdown_status.yml
```

### set_spectre_meltdown_flags.yml
Use this playbook to set the three flags for the spectre meltdown protection.

```
ansible-playbook -i hosts set_spectre_meltdown_flags.yml
```

These playbooks are software development tool and are not supported.

The Red Hat Customer Portal provides an official
[Spectre And Meltdown Detector](https://access.redhat.com/labs/speculativeexecution/)
to help detect if your systems are vulnerable to these CVEs.
