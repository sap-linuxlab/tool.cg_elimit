# tool.cg_elimit
Retrieves the effective value of a cgroup control for a given cgroup or process.

> :exclamation: In SLES and SLES for SAP Applications 15 SP6 `EffectiveMemoryMax`, `EffectiveMemoryHigh` and `EffectiveTasksMax` are available to get the effective limits for a cgroup directly. 

## Introduction

Getting the configured value for a specific cgroup control is easy. You can read either the file directly or using the appropriate `systemctl` command:

     # cat /sys/fs/cgroup/memory/system.slice/cron.service/memory.limit_in_bytes
    209715200

    # systemctl show -p MemoryLimit cron.service 
    MemoryLimit=209715200

But this is not necessarily the value that is applied to your process. Limits of parent cgroups might apply stronger restrictions. To get this *effective value*, this program is for.

    # cg_elimit memory.limit_in_bytes /system.slice/cron.service
    99999744

## How the effective limit is calculated?

Calculating the effective limit is fairly simple.
Start with the child cgroup you want the effective limit for and read the value of the control file.
Now compare this value with the value of the same control of the parent cgroup and take the lesser one.
Repeat this until you have reached the root cgroup and you have the effective value. 
If a control file is missing during traversal, you can ignore this and keep calculating. Typical reasons for that are:

- the control hasn't been configured,
- you are in the root cgroup or
- somewhere in the hierarchy the delegation stopped.


## Usage

To get a full help, run: `cg_elimit -h`

To get a list of the supported controls (limits), run: `cg_elimit -l`

To get the effective value of a control for a specific cgroup run: `cg_elimit CONTROL CGROUP`
If you want to see the details of the calculation add `-v`


> :exclamation: The name of the cgroup should either the absolute path (e.g. "/sys/fs/cgroup/memory/system.slice/cron.service") or the relative path starting from the cgroup mount point (e.g. "/system.slice/cron.service"). In any case, the path has to start with a slash and don't add the control file!

To get the effective value of a control for a process run: `cg_elimit CONTROL PID`
If you want to see the details of the calculation add `-v`

Per default only the retrieved value is printed to stdout. If the control is missing, nothing is returned.

## Examples

Getting the value of `memory.limit_in_bytes` for a specific process: 

    # cg_elimit memory.limit_in_bytes 5060
    99999744

Getting the value of `memory.limit_in_bytes` for a cgroup (relative to the mount point of the controller `/sys/fs/cgroup/memory`): 

    # cg_elimit memory.limit_in_bytes /system.slice/cron.service
    99999744

Getting the value of `memory.limit_in_bytes` for a cgroup (absolute path): 

    # cg_elimit memory.limit_in_bytes /sys/fs/cgroup/memory/system.slice/cron.service
    99999744

Asking for a non-existing cgroup (correct writing is important!):

    # cg_elimit memory.limit_in_bytes memory/system.slice/cron.service
    Cgroup directory "/sys/fs/cgroup/memorymemory/system.slice/cron.service" does not exist.

Using an unsupported control:

    # cg_elimit  memory.low 5060
    "memory.low" is not supported.

Using a control which has not been set up.

    # cg_elimit memory.memsw.limit_in_bytes /system.slice/cron.service 

If you want to know whats going on

    # cg_elimit -v memory.memsw.limit_in_bytes /system.slice/cron.service 
    Cgroupfs mounted at: /sys/fs/cgroup/memory
    Path to cgroup: /sys/fs/cgroup/memory/system.slice/cron.service
    /sys/fs/cgroup/memory/system.slice/cron.service/memory.memsw.limit_in_bytes missing
    Current limit: None
    /sys/fs/cgroup/memory/system.slice/memory.memsw.limit_in_bytes missing
    Current limit: None
    /sys/fs/cgroup/memory/memory.memsw.limit_in_bytes missing
    Current limit: None
    /sys/fs/cgroup/memory.memsw.limit_in_bytes missing
    Current limit: None


## Future Plans

- Support more limits.
- Add support for protections as well (not that simple)
- What else users come up with.


## Changelog

|                |               |      |
| -------------: | :-----------: | :--- |
|15.11.2022 | v0.1 | First code.        |
|16.11.2022 | v0.2 | Release on GitHub  |


## Bugs

Certainly, but not yet found.

