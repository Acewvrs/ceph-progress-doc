# ceph-progress-doc
This is the document I'm keeping to record the progress I made on Ceph.

IBM coordinator: Laura Flores

### 9/20 - 9/24

Created milestones for our project and worked on troubleshooting regarding setting up dev enviornments (VPN, Virtual Machines, etc)

Project 3 Milestones
* Week 1: Set up dev environment (Read up on background information and code)
* Week 2: Get familiar with commands & running test clusters
* Week 3-4: Brainstorming a solution in C++
* Week 5-6: First draft of the implementation
* Week 7-8: Refine implementation
* Week 9-10: Raise a PR & Review & Poster

Asked questions, started brainstorming/studying code

### 9/28 - 10/8

More troubleshooting!
Steps to pair programming using remote branch:
1. I was appointed to be the "branch leader", i.e. create a branch and push to their forked repo
2. Next, make sure that all group members are added as "collaborators" to that person's ceph fork
3. Next, run these commands to add that person's fork and fetch their branch:
4. git remote add <name of person's fork> <url to their fork> (git remote add ljflores git@github.com:ljflores/ceph.git)
5. git fetch <name of person's fork> (git fetch ljflores)
6. git checkout --track <name of person's fork>/<name of branch> (git checkout --track ljflores/wip-new-project-branch)
7. Now, you're set to commit. Before you push your changes, make sure your branch is up to date with the latest commits:
8. git pull <name of person's fork> <name of person's branch> --rebase (git pull ljflores wip-new-project-branch --rebase)
9. Now you can push (git push <name of person's fork> <name of person's branch> --> git push ljflores wip-new-project-branch

### 10/11 - 10/18
**Most of these notes were taken during this period, but I did add more later.**

Project 3 Notes

## Testing
To test your code after making changes:
1. If you're running a test cluster (executed ../src/vstart.sh), terminate it 
2. Run ninja again
3. Restart your test cluster and run the commands again
4. In ~/ceph/build run ./bin/ceph mgr module ls -f json-pretty

## Overview of the code:
+ https://github.com/Acewvrs/ceph/blob/900fb5083740b52fc19cb09ed0110b216ca59a78/src/mvon/MgrMonitor.cc#L1022C8-L1030C28   
+ Line 1022: accesses the enabled module stored as an array and store them in map.modules 
+ Line 1023-1029: 
    - iterates through each module; in each loop:
    - map.get_always_on_modules().count(p) > 0 checks to see if module 'p' is an "always on" module; we skip printing module info in this case
    - f->dump_string("module", p) prints the module p and its attributes in formatted texts
+ dump = print
+ dump_string = print the string
+ if (map.get_always_on_modules().count(p) == 0) means get the entire list of 'get always' modules, count how many p's are in it, then compare
+ m_ss is the output stream for ceph.
+ Usage:
```
m_ss << string;
```
+ So We can try:
```
for (auto& p : map.modules) {
    if (map.get_always_on_modules().count(p) > 0)
        continue;
    // We only show the name for enabled modules.  The any errors
    // etc will show up as a health checks.
    // f->dump_string("module", p);
    p.dump(f.get());
    m_ss << "hello! \n";
}
```


## Brainstorming:
* Replace line 1028 with p.dump(f.get()), where p is the moduleinfo (MgrMap::ModuleInfo)
* The condition that is "always on" currently continues; maybe make a separate dump_string based on disabled output to iterate through modules and output their details using a similar approach 
* need to extend data structure used for enabled and always-on modules to include additional information like health, description, version, etc. 
* set module = map.available_modules.at(p) (value in the loop to get values rather than just continue?)
* module.dump(f.get());
* loop through the always_on_modules and then if it is not the end of the loop, use a pointer it->second.dump Will test it->second.dump(&f) and it->second.dump(f.get())
* Store MgrMap::ModuleInfo somewhere, loop through & filter out always_enabled, enabled, and disabled modules
* Store always_on_modules to optimize printing detailed info for enabled modules

## Useful Notes:
* p.dump is defined [here](https://github.com/Acewvrs/ceph/blob/7e3f09e6651db50de5d5eb43c76071534d0617b9/src/mon/MgrMap.h#L555C3-L555C38)
* Difference between p.dump(f.get()) and f->dump_string("module", p)
* Unit test: src/test/mgr

## Questions:
* It seems like we want our output to look like the one from p.dump(f.get()). Can we directly change the dump_string() to make it print full details? Or should we stick with p.dump()?
* Should we make a separate dump() for the always - on 
* How is f (pointer) defined? What is a scoped pointer in the boost library?

### 10/22 - 10/26
* Worked on code
* Merging multiple commits made by teammates (and me)

git fetch <Jun's fork> <br />
git pull <Jun's fork> <branch name> --rebase (this will incorporate your project mates' changes without overwriting your commits)

**Optimization Idea:**
* Use hashmap instead of std::map
* Better run time (on average) for storeing and getting modules!
+ **Caveat: hashmap stores its elements in a different order! Thus, the order is different when iterating through the elements. Unless...**

**Test and Confirm**
* I tested and there's no difference in output. I think it makes sense because the only difference between std::map and std::unordered_map is their runtime
* The order in which the keys are stored is different. But it won't affect the order of the modules when they are printed because we're iterating through a different container
[here]( https://github.com/Acewvrs/ceph/blob/632cb041e7234573bec769a2b0b092ad62380a55/src/mon/MgrMonitor.cc#L1025).
* You can see that when printing the "always enabled" modules, we're traversing the keys stored in a map (map.get_always_on_modules()), and not the unordered_map (module_info_map) we created. So the order these modules are printed is based on how they are stored in map.get_always_on_modules(). And because it's a map, the keys are sorted alphabetically.
* Full output is attached at the bottom of this documentation
 
### 10/29
Document everything & progress we made

At this point, we're ready to commit our work! 

Read the PR guideline for Ceph, which is [here](https://github.com/ceph/ceph/blob/main/SubmittingPatches.rst). <br />
I ensured our commit titles and the content of PR were adhering to the guidelines. <br />
After everything looked good, I raised a [pull request](https://github.com/ceph/ceph/pull/60548)

### 11/01
Currently, our PR fails checks...
Most of those come from the merge Conflict

Went over how to  resolve git conflicts:
- This can happen when you raise a pull request and the github bot adds the "needs-rebase" label
- How do I resolve this?
- Check out your branch
- Make sure it's up to date with the pull request
- run 'git status' to check which file(s) have conflicts (these will be red)
- open the file and search for "HEAD"
- incorporate your changes in with the conflicted changes
- commit your changes
- Add a conflicts section (only do this for backports; fine to leave out on the main branch)

Problem: encoutering the following error when git pull
error: insufficient permission for adding an object to repository database .git/objects
solution: run it with sudo

When pushing:
We need to use -f when pushing because we rebased

### 11/03
Started working on another issue: https://tracker.ceph.com/issues/67812

Task: Implement a separate Ceph command that gathers new stretch cluster data, which the following info.
"stretch_mode": {
    "stretch_mode_enabled": false,
    "stretch_bucket_count": 0,
    "degraded_stretch_mode": 0,
    "recovering_stretch_mode": 0,
    "stretch_mode_bucket": 0
}

## Brainstorming:
* Study the code we worked on. Except the command 'ceph mgr module stretch" to get the stretch cluster data
* This block of code gives us some insight into how we can use osdmap.
```
if (mon.osdmon()->osdmap.get_num_osds() > 0 &&
       now > mon.monmap->created + g_conf().get_val<int64_t>("mon_mgr_mkfs_grace")) {
    health_status_t level = HEALTH_WARN;
    if (first_seen_inactive != utime_t() &&
	now - first_seen_inactive > g_conf().get_val<int64_t>("mon_mgr_inactive_grace")) {
      level = HEALTH_ERR;
    }
    return level;
  }
```
* It seems like we can directly access everything we need from the osdmap.
* Next question: how do I print it?
* osdmap provides dump(ceph::Formatter *f) function that handles printing. In our case, f is already provided.


## Useful Notes
* osdmap is defined [here](https://github.com/Acewvrs/ceph/blob/jun-debug/src/osd/OSDMap.h)
* The following variables are public:
    - stretch_mode_enabled(false),
    - stretch_bucket_count(0),
    - degraded_stretch_mode(0),
    - recovering_stretch_mode(0),
    - stretch_mode_bucket(0)

### 11/05 
Resolved: merge conflict and "sign-off" error.

Problem: our PR has an API issue and fails [one of the tests](https://github.com/Acewvrs/ceph/blob/5acf3d01c18f7bc43792ead1c5000e3e2d671a1f/qa/tasks/mgr/mgr_test_case.py#L111)
TypeError: unhashable type: 'dict'

The variable loaded is a list of dictionaries rather than a list of strings or other hashable types (as can be seen from the output below). When you attempt to create a set from loaded, Python throws an error because it cannot hash dictionaries.

Intuiton: this is because we switched to using the dump() function from the dump_string() function:
it->second->dump(f.get())
f->dump_string("module", p);

The dump_string seems to be the more proper way to print out, given that it takes the "module" option. 

THE ABOVE SOLUTION IS WRONG because... 

Previously, the enabled_module list only displayed the names of the modules like this: 

"enabled_modules": ["iostat", "nfs"] 

Each item in the list was a string, which is a hashable type and what the set() function is expecting. However, because we wanted our enabled modules to print some other details, it now shows up like this:

"enabled_modules": [  { "name": "iostat", "can_run": true }, {...}, {...} ], where each item is now a dictionary, which is not hashable. I believe the only way around this is to edit the test file (mgr_test_case.py) directly so that it correctly parses the new output format, like so:

module_names = {module['name'] for module in loaded}
unload_modules = module_names - {"cephadm", "restful"}

This would still be an O(n) operation to add the name of each enabled module to create a set of size n, and the overall time complexity would remain the same.

For reference to the error [click here](https://github.com/Acewvrs/ceph/blob/5acf3d01c18f7bc43792ead1c5000e3e2d671a1f/qa/tasks/mgr/mgr_test_case.py#L111)

### 11/07
The above issue has been fixed!

However, we've got another issue: the API test is timing out at [this line](https://github.com/Acewvrs/ceph/blob/5acf3d01c18f7bc43792ead1c5000e3e2d671a1f/qa/tasks/mgr/mgr_test_case.py#L196)

Intuition: This problem is a continuation of the above (11/5) issue.
It seems like there's multiple cases of getting the enabled modules' names, all of which are handling the returned set wrong as now it is returning a set of dictionary instead of strings.
So once I correctly handle the new returned type using my solution above.

### 11/08
I notified Laura on the timeout error.
I raised the issue on the official Ceph issue tracker.
https://tracker.ceph.com/issues/68883

### 11/12
The timeout error has been resolved!

Right now, I'm just waiting for our code to be reviewed.

I demonstrated how our project made an impact by showing the differences in output for the 'mgr module ls -f json-pretty' command.

In the meantime, I'll work on the issue I found on 11/3.

### 11/15-11/17
I realized someone else was already assigned to the issue I found on Nov. 3.

I'll start working on a new issue [here](https://tracker.ceph.com/issues/68892)

## Brainstorming
The error is caused here:
```
    if (!force && !pending_map.can_run_module(module, &can_run_error)) {
      ss << "module '" << module << "' reports that it cannot run on the active "
            "manager daemon: " << can_run_error << " (pass --force to force "
            "enablement)";
      r = -ENOENT;
      goto out;
    }
```
can_run_module is defined [here](https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mon/MgrMap.h#L355).

* Based on the implementation. The "influx" module's "can_run" attribute is set to false, and this is causing the can_run_module function to return false.
* Look at where the module is created first and set the can_run to true.
* But can_run is set to true [by default](https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mon/MgrMap.h#L130), which means this variable was later set to false for this module... why?
* use 'grep -R "can_run = false" src/' in build directory to find exactly where we're setting the variable
* can_run is set [here] (https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mgr/MgrStandby.cc#L228)
* get_modules is defined [here] (https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mgr/PyModuleRegistry.h#L82C8-L82C19)
* pyModuleRef is defined [here] (https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mgr/PyModule.h#L174)
* and can_run is set to false in pyModule [here] (https://github.com/Acewvrs/ceph/blob/bfa057d9fcd570ed2944817c2172998eb1b75235/src/mgr/PyModule.h#L68)

### 11/19
One of the contributors left comments/suggestions/feedback on our PR:
* Indentation error (easy fix)
* Block - instead of using a block, use a separate function. (easy fix)
* You are manipulating pointers into a vector, if I'm not mistaken. I do not know the locking infra for the modules. Are we sure there is no possibility of change to the modules vector by another thread during the execution of this function? (hard)

I worked on resolving these issues.

## Braninstorming
* There doesn't seem to be any mutex locks for the ModuleInfo class, so I think it is possible for the class to be modified by another thread. I think one way to get around this vulnerability is instead of directly storing the ModuleInfo pointers, we can store only the module names as strings, and grab its info using the get_module_info function when we need them: https://github.com/Acewvrs/ceph/blob/fb2877ccb6f9d58f59e0577ee75d98e196ef595f/src/mon/MgrMap.h#L349
* map is of type MgrMap, and is set [here] (https://github.com/Acewvrs/ceph/blob/fb2877ccb6f9d58f59e0577ee75d98e196ef595f/src/mon/MgrMonitor.h#L28)

### 11/20
I received more feedback from contributors on our PR:
* Small commit message fixes
* Remove container that stores module names and use std::find_if
* Drop two commits that, when formed together, do nothing
  
## Notes for dropping commits:
I can easily drop commits by following these steps:
1. git checkout my-pull-request-branch
2. git rebase -i HEAD~n // where n is the number of last commits you want to include in interactive rebase.
3. Replace pick with drop for commits you want to discard.
4. Save and exit.
5. git push songj project3 --force-with-lease

## Brainstorm for removing container
* available_modules is a vector of ModuleInfo (std::vector<ModuleInfo>)
* each ModuleInfo has a name variable
* instead of creating a separate container like [here] (https://github.com/Acewvrs/ceph/blob/f074573b8973b4da7cc7f613b8a079fe37cded00/src/mon/MgrMonitor.cc#L1018C7-L1018C68), we can do something like std::find_if(map.available_modules.begin(), map.available_modules.end(), [](const ModuleInfo & m) -> bool { return m.name == module_name; });
* save the returned iterator like: auto module_itr = std::find_if(...);
* then use it like *module_itr;
* if there's no module that satisfies the condition, it returns map.available_modules.end().

## Question:
* I was under the impression that the "conflict" section is required for merge conflicts. Should I keep the section, or remove it?

I worked on fixing these issues.

### 11/22
I worked on the poster for our presentation day (12/3) 

The poster slide is [here](https://docs.google.com/presentation/d/1UjsYCNjg86qSgsUsJZQYNDXcKvA6p98O7o98L6UmOvk/edit?usp=sharing)

### 11/26
I continued to work on the poster.

### Output
Full output of the command 'ceph mgr module ls ls -f json-pretty':
<details>  
{
    "always_on_modules": [
        {
            "name": "balancer",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "active": {
                    "name": "active",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "automatically balance PGs across cluster",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "begin_time": {
                    "name": "begin_time",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "0000",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "beginning time of day to automatically balance",
                    "long_desc": "This is a time of day in the format HHMM.",
                    "tags": [],
                    "see_also": []
                },
                "begin_weekday": {
                    "name": "begin_weekday",
                    "type": "uint",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "0",
                    "min": "0",
                    "max": "6",
                    "enum_allowed": [],
                    "desc": "Restrict automatic balancing to this day of the week or later",
                    "long_desc": "0 = Sunday, 1 = Monday, etc.",
                    "tags": [],
                    "see_also": []
                },
                "crush_compat_max_iterations": {
                    "name": "crush_compat_max_iterations",
                    "type": "uint",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "25",
                    "min": "1",
                    "max": "250",
                    "enum_allowed": [],
                    "desc": "maximum number of iterations to attempt optimization",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "crush_compat_metrics": {
                    "name": "crush_compat_metrics",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "pgs,objects,bytes",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "metrics with which to calculate OSD utilization",
                    "long_desc": "Value is a list of one or more of \"pgs\", \"objects\", or \"bytes\", and indicates which metrics to use to balance utilization.",
                    "tags": [],
                    "see_also": []
                },
                "crush_compat_step": {
                    "name": "crush_compat_step",
                    "type": "float",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "0.5",
                    "min": "0.001",
                    "max": "0.999",
                    "enum_allowed": [],
                    "desc": "aggressiveness of optimization",
                    "long_desc": ".99 is very aggressive, .01 is less aggressive",
                    "tags": [],
                    "see_also": []
                },
                "end_time": {
                    "name": "end_time",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "2359",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "ending time of day to automatically balance",
                    "long_desc": "This is a time of day in the format HHMM.",
                    "tags": [],
                    "see_also": []
                },
                "end_weekday": {
                    "name": "end_weekday",
                    "type": "uint",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "0",
                    "min": "0",
                    "max": "6",
                    "enum_allowed": [],
                    "desc": "Restrict automatic balancing to days of the week earlier than this",
                    "long_desc": "0 = Sunday, 1 = Monday, etc.",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "min_score": {
                    "name": "min_score",
                    "type": "float",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "minimum score, below which no optimization is attempted",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "mode": {
                    "name": "mode",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "upmap",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "crush-compat",
                        "none",
                        "read",
                        "upmap",
                        "upmap-read"
                    ],
                    "desc": "Balancer mode",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "pool_ids": {
                    "name": "pool_ids",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "pools which the automatic balancing will be limited to",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sleep_interval": {
                    "name": "sleep_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "60",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how frequently to wake up and attempt optimization",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "upmap_max_deviation": {
                    "name": "upmap_max_deviation",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "5",
                    "min": "1",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "deviation below which no optimization is attempted",
                    "long_desc": "If the number of PGs are within this count then no optimization is attempted",
                    "tags": [],
                    "see_also": []
                },
                "upmap_max_optimizations": {
                    "name": "upmap_max_optimizations",
                    "type": "uint",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "maximum upmap optimizations to make per attempt",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "crash",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "retain_interval": {
                    "name": "retain_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "31536000",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how long to retain crashes before pruning them",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "warn_recent_interval": {
                    "name": "warn_recent_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "1209600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "time interval in which to warn about recent crashes",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "devicehealth",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "enable_monitoring": {
                    "name": "enable_monitoring",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "monitor device health metrics",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "mark_out_threshold": {
                    "name": "mark_out_threshold",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "2419200",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "automatically mark OSD if it may fail before this long",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "pool_name": {
                    "name": "pool_name",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "device_health_metrics",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "name of pool in which to store device health metrics",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "retention_period": {
                    "name": "retention_period",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "15552000",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how long to retain device health metrics",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "scrape_frequency": {
                    "name": "scrape_frequency",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "86400",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how frequently to scrape device health metrics",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "self_heal": {
                    "name": "self_heal",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "preemptively heal cluster around devices that may fail",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sleep_interval": {
                    "name": "sleep_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how frequently to wake up and check device health",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "warn_threshold": {
                    "name": "warn_threshold",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "7257600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "raise health warning if OSD may fail before this long",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "orchestrator",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "fail_fs": {
                    "name": "fail_fs",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Fail filesystem for rapid multi-rank mds upgrade",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "orchestrator": {
                    "name": "orchestrator",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "cephadm",
                        "rook",
                        "test_orchestrator"
                    ],
                    "desc": "Orchestrator backend",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "pg_autoscaler",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sleep_interval": {
                    "name": "sleep_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "60",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "threshold": {
                    "name": "threshold",
                    "type": "float",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "3.0",
                    "min": "1.0",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "scaling threshold",
                    "long_desc": "The factor by which the `NEW PG_NUM` must vary from the current`PG_NUM` before being accepted. Cannot be less than 1.0",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "progress",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "allow_pg_recovery_event": {
                    "name": "allow_pg_recovery_event",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "allow the module to show pg recovery progress",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "enabled": {
                    "name": "enabled",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_completed_events": {
                    "name": "max_completed_events",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "50",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "number of past completed events to remember",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sleep_interval": {
                    "name": "sleep_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "5",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how long the module is going to sleep",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "rbd_support",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_concurrent_snap_create": {
                    "name": "max_concurrent_snap_create",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "mirror_snapshot_schedule": {
                    "name": "mirror_snapshot_schedule",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "trash_purge_schedule": {
                    "name": "trash_purge_schedule",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "status",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "telemetry",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "channel_basic": {
                    "name": "channel_basic",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Share basic cluster information (size, version)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "channel_crash": {
                    "name": "channel_crash",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Share metadata about Ceph daemon crashes (version, stack straces, etc)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "channel_device": {
                    "name": "channel_device",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Share device health metrics (e.g., SMART data, minus potentially identifying info like serial numbers)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "channel_ident": {
                    "name": "channel_ident",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Share a user-provided description and/or contact email for the cluster",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "channel_perf": {
                    "name": "channel_perf",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Share various performance metrics of a cluster",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "contact": {
                    "name": "contact",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "description": {
                    "name": "description",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "device_url": {
                    "name": "device_url",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "https://telemetry.ceph.com/device",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "enabled": {
                    "name": "enabled",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "interval": {
                    "name": "interval",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "24",
                    "min": "8",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "last_opt_revision": {
                    "name": "last_opt_revision",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "1",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "leaderboard": {
                    "name": "leaderboard",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "leaderboard_description": {
                    "name": "leaderboard_description",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "organization": {
                    "name": "organization",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "proxy": {
                    "name": "proxy",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "url": {
                    "name": "url",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "https://telemetry.ceph.com/report",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "volumes",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_concurrent_clones": {
                    "name": "max_concurrent_clones",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "4",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Number of asynchronous cloner threads",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "periodic_async_work": {
                    "name": "periodic_async_work",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Periodically check for async work",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "snapshot_clone_delay": {
                    "name": "snapshot_clone_delay",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Delay clone begin operation by snapshot_clone_delay seconds",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "snapshot_clone_no_wait": {
                    "name": "snapshot_clone_no_wait",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Reject subvolume clone request when cloner threads are busy",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        }
    ],
    "enabled_modules": [
        {
            "name": "iostat",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "nfs",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        }
    ],
    "disabled_modules": [
        {
            "name": "alerts",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "interval": {
                    "name": "interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "60",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "How frequently to reexamine health status",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_destination": {
                    "name": "smtp_destination",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Email address to send alerts to, use commas to separate multiple",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_from_name": {
                    "name": "smtp_from_name",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "Ceph",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Email From: name",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_host": {
                    "name": "smtp_host",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "SMTP server",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_password": {
                    "name": "smtp_password",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Password to authenticate with",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_port": {
                    "name": "smtp_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "465",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "SMTP port",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_sender": {
                    "name": "smtp_sender",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "SMTP envelope sender",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_ssl": {
                    "name": "smtp_ssl",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Use SSL to connect to SMTP server",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "smtp_user": {
                    "name": "smtp_user",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "User to authenticate as",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "cephadm",
            "can_run": false,
            "error_string": "loading asyncssh library:No module named 'asyncssh'",
            "module_options": {
                "agent_down_multiplier": {
                    "name": "agent_down_multiplier",
                    "type": "float",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "3.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Multiplied by agent refresh rate to calculate how long agent must not report before being marked down",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "agent_refresh_rate": {
                    "name": "agent_refresh_rate",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "20",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "How often agent on each host will try to gather and send metadata",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "agent_starting_port": {
                    "name": "agent_starting_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "4721",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "First port agent will try to bind to (will also try up to next 1000 subsequent ports if blocked)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "allow_ptrace": {
                    "name": "allow_ptrace",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "allow SYS_PTRACE capability on ceph containers",
                    "long_desc": "The SYS_PTRACE capability is needed to attach to a process with gdb or strace.  Enabling this options can allow debugging daemons that encounter problems at runtime.",
                    "tags": [],
                    "see_also": []
                },
                "autotune_interval": {
                    "name": "autotune_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how frequently to autotune daemon memory",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "autotune_memory_target_ratio": {
                    "name": "autotune_memory_target_ratio",
                    "type": "float",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "0.7",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "ratio of total system memory to divide amongst autotuned daemons",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "cephadm_log_destination": {
                    "name": "cephadm_log_destination",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "file",
                        "file,syslog",
                        "syslog"
                    ],
                    "desc": "Destination for cephadm command's persistent logging",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "cgroups_split": {
                    "name": "cgroups_split",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Pass --cgroups=split when cephadm creates containers (currently podman only)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "config_checks_enabled": {
                    "name": "config_checks_enabled",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Enable or disable the cephadm configuration analysis",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "config_dashboard": {
                    "name": "config_dashboard",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "manage configs like API endpoints in Dashboard.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_alertmanager": {
                    "name": "container_image_alertmanager",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/prometheus/alertmanager:v0.27.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Prometheus container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_base": {
                    "name": "container_image_base",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "quay.io/ceph/ceph",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Container image name, without the tag",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_elasticsearch": {
                    "name": "container_image_elasticsearch",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/omrizeneva/elasticsearch:6.8.23",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "elasticsearch container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_grafana": {
                    "name": "container_image_grafana",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/grafana:10.4.8",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Prometheus container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_haproxy": {
                    "name": "container_image_haproxy",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/haproxy:2.3",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "HAproxy container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_jaeger_agent": {
                    "name": "container_image_jaeger_agent",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/jaegertracing/jaeger-agent:1.29",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Jaeger agent container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_jaeger_collector": {
                    "name": "container_image_jaeger_collector",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/jaegertracing/jaeger-collector:1.29",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Jaeger collector container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_jaeger_query": {
                    "name": "container_image_jaeger_query",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/jaegertracing/jaeger-query:1.29",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Jaeger query container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_keepalived": {
                    "name": "container_image_keepalived",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/keepalived:2.2.4",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Keepalived container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_loki": {
                    "name": "container_image_loki",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/loki:3.0.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Loki container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_nginx": {
                    "name": "container_image_nginx",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/nginx:sclorg-nginx-126",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Nginx container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_node_exporter": {
                    "name": "container_image_node_exporter",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/prometheus/node-exporter:v1.7.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Prometheus container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_nvmeof": {
                    "name": "container_image_nvmeof",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/nvmeof:1.2.17",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Nvme-of container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_oauth2_proxy": {
                    "name": "container_image_oauth2_proxy",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/oauth2-proxy/oauth2-proxy:v7.6.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "oauth2-proxy container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_prometheus": {
                    "name": "container_image_prometheus",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/prometheus/prometheus:v2.51.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Prometheus container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_promtail": {
                    "name": "container_image_promtail",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/promtail:3.0.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Promtail container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_samba": {
                    "name": "container_image_samba",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/samba.org/samba-server:devbuilds-centos-amd64",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Samba/SMB container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_samba_metrics": {
                    "name": "container_image_samba_metrics",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/samba.org/samba-metrics:latest",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Samba/SMB metrics exporter container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_image_snmp_gateway": {
                    "name": "container_image_snmp_gateway",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io/ceph/snmp-notifier:v1.2.1",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "SNMP Gateway container image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "container_init": {
                    "name": "container_init",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Run podman/docker with `--init`",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "daemon_cache_timeout": {
                    "name": "daemon_cache_timeout",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "seconds to cache service (daemon) inventory",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "default_cephadm_command_timeout": {
                    "name": "default_cephadm_command_timeout",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "900",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Default timeout applied to cephadm commands run directly on the host (in seconds)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "default_registry": {
                    "name": "default_registry",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "quay.io",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Search-registry to which we should normalize unqualified image names. This is not the default registry",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "device_cache_timeout": {
                    "name": "device_cache_timeout",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "1800",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "seconds to cache device inventory",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "device_enhanced_scan": {
                    "name": "device_enhanced_scan",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Use libstoragemgmt during device scans",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "facts_cache_timeout": {
                    "name": "facts_cache_timeout",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "60",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "seconds to cache host facts data",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "grafana_dashboards_path": {
                    "name": "grafana_dashboards_path",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "/etc/grafana/dashboards/ceph-dashboard/",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "location of dashboards to include in grafana deployments",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "host_check_interval": {
                    "name": "host_check_interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "how frequently to perform a host check",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "hw_monitoring": {
                    "name": "hw_monitoring",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Deploy hw monitoring daemon on every host.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "inventory_list_all": {
                    "name": "inventory_list_all",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Whether ceph-volume inventory should report more devices (mostly mappers (LVs / mpaths), partitions...)",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_refresh_metadata": {
                    "name": "log_refresh_metadata",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Log all refresh metadata. Includes daemon, device, and host info collected regularly. Only has effect if logging at debug level",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "log to the \"cephadm\" cluster log channel\"",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "manage_etc_ceph_ceph_conf": {
                    "name": "manage_etc_ceph_ceph_conf",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Manage and own /etc/ceph/ceph.conf on the hosts.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "manage_etc_ceph_ceph_conf_hosts": {
                    "name": "manage_etc_ceph_ceph_conf_hosts",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "*",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "PlacementSpec describing on which hosts to manage /etc/ceph/ceph.conf",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_count_per_host": {
                    "name": "max_count_per_host",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "max number of daemons per service per host",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_osd_draining_count": {
                    "name": "max_osd_draining_count",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "max number of osds that will be drained simultaneously when osds are removed",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "migration_current": {
                    "name": "migration_current",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "internal - do not modify",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "mode": {
                    "name": "mode",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "root",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "cephadm-package",
                        "root"
                    ],
                    "desc": "mode for remote execution of cephadm",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "oob_default_addr": {
                    "name": "oob_default_addr",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "169.254.1.1",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Default address for RedFish API (oob management).",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "prometheus_alerts_path": {
                    "name": "prometheus_alerts_path",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "/etc/prometheus/ceph/ceph_default_alerts.yml",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "location of alerts to include in prometheus deployments",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "registry_insecure": {
                    "name": "registry_insecure",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Registry is to be considered insecure (no TLS available). Only for development purposes.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "registry_password": {
                    "name": "registry_password",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Custom repository password. Only used for logging into a registry.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "registry_url": {
                    "name": "registry_url",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Registry url for login purposes. This is not the default registry",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "registry_username": {
                    "name": "registry_username",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Custom repository username. Only used for logging into a registry.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "secure_monitoring_stack": {
                    "name": "secure_monitoring_stack",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Enable TLS security for all the monitoring stack daemons",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "service_discovery_port": {
                    "name": "service_discovery_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "8765",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "cephadm service discovery port",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssh_config_file": {
                    "name": "ssh_config_file",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "customized SSH config file to connect to managed hosts",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssh_keepalive_count_max": {
                    "name": "ssh_keepalive_count_max",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "3",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "How many times ssh connections can fail liveness checks before the host is marked offline",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssh_keepalive_interval": {
                    "name": "ssh_keepalive_interval",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "7",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "How often ssh connections are checked for liveness",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "use_agent": {
                    "name": "use_agent",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Use cephadm agent on each host to gather and send metadata",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "use_repo_digest": {
                    "name": "use_repo_digest",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Automatically convert image tags to image digest. Make sure all daemons use the same image",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "warn_on_failed_host_check": {
                    "name": "warn_on_failed_host_check",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "raise a health warning if the host check fails",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "warn_on_stray_daemons": {
                    "name": "warn_on_stray_daemons",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "raise a health warning if daemons are detected that are not managed by cephadm",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "warn_on_stray_hosts": {
                    "name": "warn_on_stray_hosts",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "raise a health warning if daemons are detected on a host that is not managed by cephadm",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "cli_api",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "dashboard",
            "can_run": false,
            "error_string": "Frontend assets not found at '/home/songj/Documents/ceph/build/src/pybind/mgr/dashboard/frontend/dist': incomplete build?",
            "module_options": {
                "ACCOUNT_LOCKOUT_ATTEMPTS": {
                    "name": "ACCOUNT_LOCKOUT_ATTEMPTS",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ALERTMANAGER_API_HOST": {
                    "name": "ALERTMANAGER_API_HOST",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ALERTMANAGER_API_SSL_VERIFY": {
                    "name": "ALERTMANAGER_API_SSL_VERIFY",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "AUDIT_API_ENABLED": {
                    "name": "AUDIT_API_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "AUDIT_API_LOG_PAYLOAD": {
                    "name": "AUDIT_API_LOG_PAYLOAD",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ENABLE_BROWSABLE_API": {
                    "name": "ENABLE_BROWSABLE_API",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_CEPHFS": {
                    "name": "FEATURE_TOGGLE_CEPHFS",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_DASHBOARD": {
                    "name": "FEATURE_TOGGLE_DASHBOARD",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_ISCSI": {
                    "name": "FEATURE_TOGGLE_ISCSI",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_MIRRORING": {
                    "name": "FEATURE_TOGGLE_MIRRORING",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_NFS": {
                    "name": "FEATURE_TOGGLE_NFS",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_RBD": {
                    "name": "FEATURE_TOGGLE_RBD",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "FEATURE_TOGGLE_RGW": {
                    "name": "FEATURE_TOGGLE_RGW",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GANESHA_CLUSTERS_RADOS_POOL_NAMESPACE": {
                    "name": "GANESHA_CLUSTERS_RADOS_POOL_NAMESPACE",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_API_PASSWORD": {
                    "name": "GRAFANA_API_PASSWORD",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "admin",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_API_SSL_VERIFY": {
                    "name": "GRAFANA_API_SSL_VERIFY",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_API_URL": {
                    "name": "GRAFANA_API_URL",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_API_USERNAME": {
                    "name": "GRAFANA_API_USERNAME",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "admin",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_FRONTEND_API_URL": {
                    "name": "GRAFANA_FRONTEND_API_URL",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "GRAFANA_UPDATE_DASHBOARDS": {
                    "name": "GRAFANA_UPDATE_DASHBOARDS",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ISCSI_API_SSL_VERIFICATION": {
                    "name": "ISCSI_API_SSL_VERIFICATION",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ISSUE_TRACKER_API_KEY": {
                    "name": "ISSUE_TRACKER_API_KEY",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "MANAGED_BY_CLUSTERS": {
                    "name": "MANAGED_BY_CLUSTERS",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "[]",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "MULTICLUSTER_CONFIG": {
                    "name": "MULTICLUSTER_CONFIG",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "{}",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PROMETHEUS_API_HOST": {
                    "name": "PROMETHEUS_API_HOST",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PROMETHEUS_API_SSL_VERIFY": {
                    "name": "PROMETHEUS_API_SSL_VERIFY",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_COMPLEXITY_ENABLED": {
                    "name": "PWD_POLICY_CHECK_COMPLEXITY_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_EXCLUSION_LIST_ENABLED": {
                    "name": "PWD_POLICY_CHECK_EXCLUSION_LIST_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_LENGTH_ENABLED": {
                    "name": "PWD_POLICY_CHECK_LENGTH_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_OLDPWD_ENABLED": {
                    "name": "PWD_POLICY_CHECK_OLDPWD_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_REPETITIVE_CHARS_ENABLED": {
                    "name": "PWD_POLICY_CHECK_REPETITIVE_CHARS_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_SEQUENTIAL_CHARS_ENABLED": {
                    "name": "PWD_POLICY_CHECK_SEQUENTIAL_CHARS_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_CHECK_USERNAME_ENABLED": {
                    "name": "PWD_POLICY_CHECK_USERNAME_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_ENABLED": {
                    "name": "PWD_POLICY_ENABLED",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_EXCLUSION_LIST": {
                    "name": "PWD_POLICY_EXCLUSION_LIST",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "osd,host,dashboard,pool,block,nfs,ceph,monitors,gateway,logs,crush,maps",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_MIN_COMPLEXITY": {
                    "name": "PWD_POLICY_MIN_COMPLEXITY",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "PWD_POLICY_MIN_LENGTH": {
                    "name": "PWD_POLICY_MIN_LENGTH",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "8",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "REST_REQUESTS_TIMEOUT": {
                    "name": "REST_REQUESTS_TIMEOUT",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "45",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "RGW_API_ACCESS_KEY": {
                    "name": "RGW_API_ACCESS_KEY",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "RGW_API_ADMIN_RESOURCE": {
                    "name": "RGW_API_ADMIN_RESOURCE",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "admin",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "RGW_API_SECRET_KEY": {
                    "name": "RGW_API_SECRET_KEY",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "RGW_API_SSL_VERIFY": {
                    "name": "RGW_API_SSL_VERIFY",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "UNSAFE_TLS_v1_2": {
                    "name": "UNSAFE_TLS_v1_2",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "USER_PWD_EXPIRATION_SPAN": {
                    "name": "USER_PWD_EXPIRATION_SPAN",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "USER_PWD_EXPIRATION_WARNING_1": {
                    "name": "USER_PWD_EXPIRATION_WARNING_1",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "USER_PWD_EXPIRATION_WARNING_2": {
                    "name": "USER_PWD_EXPIRATION_WARNING_2",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "5",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "cross_origin_url": {
                    "name": "cross_origin_url",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "crt_file": {
                    "name": "crt_file",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "debug": {
                    "name": "debug",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Enable/disable debug options",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "jwt_token_ttl": {
                    "name": "jwt_token_ttl",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "28800",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "key_file": {
                    "name": "key_file",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "motd": {
                    "name": "motd",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "The message of the day",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "redirect_resolve_ip_addr": {
                    "name": "redirect_resolve_ip_addr",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_addr": {
                    "name": "server_addr",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "::",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_port": {
                    "name": "server_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "8080",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssl": {
                    "name": "ssl",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssl_server_port": {
                    "name": "ssl_server_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "8443",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sso_oauth2": {
                    "name": "sso_oauth2",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "standby_behaviour": {
                    "name": "standby_behaviour",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "redirect",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "error",
                        "redirect"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "standby_error_status_code": {
                    "name": "standby_error_status_code",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "500",
                    "min": "400",
                    "max": "599",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "url_prefix": {
                    "name": "url_prefix",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "diskprediction_local",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "predict_interval": {
                    "name": "predict_interval",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "86400",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "predictor_model": {
                    "name": "predictor_model",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "prophetstor",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sleep_interval": {
                    "name": "sleep_interval",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "600",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "feedback",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "hello",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "emphatic": {
                    "name": "emphatic",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "whether to say it loudly",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "foo": {
                    "name": "foo",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "a",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "a",
                        "b",
                        "c"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "place": {
                    "name": "place",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "world",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "a place in the world",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "influx",
            "can_run": false,
            "error_string": "influxdb python module not found",
            "module_options": {
                "batch_size": {
                    "name": "batch_size",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "5000",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "How big batches of data points should be when sending to InfluxDB.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "database": {
                    "name": "database",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "ceph",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "InfluxDB database name. You will need to create this database and grant write privileges to the configured username or the username must have admin privileges to create it.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "hostname": {
                    "name": "hostname",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "InfluxDB server hostname",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "interval": {
                    "name": "interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "30",
                    "min": "5",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Time between reports to InfluxDB.  Default 30 seconds.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "password": {
                    "name": "password",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "password of InfluxDB server user",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "port": {
                    "name": "port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "8086",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "InfluxDB server port",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "ssl": {
                    "name": "ssl",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "false",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Use https connection for InfluxDB server. Use \"true\" or \"false\".",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "threads": {
                    "name": "threads",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "5",
                    "min": "1",
                    "max": "32",
                    "enum_allowed": [],
                    "desc": "How many worker threads should be spawned for sending data to InfluxDB.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "username": {
                    "name": "username",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "username of InfluxDB server user",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "verify_ssl": {
                    "name": "verify_ssl",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "true",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Verify https cert for InfluxDB server. Use \"true\" or \"false\".",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "insights",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "k8sevents",
            "can_run": false,
            "error_string": "kubernetes python client is not available",
            "module_options": {
                "ceph_event_retention_days": {
                    "name": "ceph_event_retention_days",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "7",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Days to hold ceph event information within local cache",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "config_check_secs": {
                    "name": "config_check_secs",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10",
                    "min": "10",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "interval (secs) to check for cluster configuration changes",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "localpool",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "failure_domain": {
                    "name": "failure_domain",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "host",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "failure domain for any created local pool",
                    "long_desc": "what failure domain we should separate data replicas across.",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "min_size": {
                    "name": "min_size",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "default min_size for any created local pool",
                    "long_desc": "value to set min_size to (unchanged from Ceph's default if this option is not set)",
                    "tags": [],
                    "see_also": []
                },
                "num_rep": {
                    "name": "num_rep",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "3",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "default replica count for any created local pool",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "pg_num": {
                    "name": "pg_num",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "128",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "default pg_num for any created local pool",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "prefix": {
                    "name": "prefix",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "name prefix for any created local pool",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "subtree": {
                    "name": "subtree",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "rack",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "CRUSH level for which to create a local pool",
                    "long_desc": "which CRUSH subtree type the module should create a pool for.",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "mds_autoscaler",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "mirroring",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "osd_perf_query",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "osd_support",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "prometheus",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "cache": {
                    "name": "cache",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "exclude_perf_counters": {
                    "name": "exclude_perf_counters",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Do not include perf-counters in the metrics output",
                    "long_desc": "Gathering perf-counters from a single Prometheus exporter can degrade ceph-mgr performance, especially in large clusters. Instead, Ceph-exporter daemons are now used by default for perf-counter gathering. This should only be disabled when no ceph-exporters are deployed.",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rbd_stats_pools": {
                    "name": "rbd_stats_pools",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rbd_stats_pools_refresh_interval": {
                    "name": "rbd_stats_pools_refresh_interval",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "300",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "scrape_interval": {
                    "name": "scrape_interval",
                    "type": "float",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "15.0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_addr": {
                    "name": "server_addr",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "::",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "the IPv4 or IPv6 address on which the module listens for HTTP requests",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_port": {
                    "name": "server_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "9283",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "the port on which the module listens for HTTP requests",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "stale_cache_strategy": {
                    "name": "stale_cache_strategy",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "log",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "standby_behaviour": {
                    "name": "standby_behaviour",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "default",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "default",
                        "error"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "standby_error_status_code": {
                    "name": "standby_error_status_code",
                    "type": "int",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "500",
                    "min": "400",
                    "max": "599",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "restful",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "enable_auth": {
                    "name": "enable_auth",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "key_file": {
                    "name": "key_file",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "max_requests": {
                    "name": "max_requests",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "500",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "Maximum number of requests to keep in memory.  When new request comes in, the oldest request will be removed if the number of requests exceeds the max request number.if un-finished request is removed, error message will be logged in the ceph-mgr log.",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_addr": {
                    "name": "server_addr",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "server_port": {
                    "name": "server_port",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "rgw",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "secondary_zone_period_retry_limit": {
                    "name": "secondary_zone_period_retry_limit",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "5",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "RGW module period update retry limit for secondary site",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "selftest",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "roption1": {
                    "name": "roption1",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "roption2": {
                    "name": "roption2",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "xyz",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption1": {
                    "name": "rwoption1",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption2": {
                    "name": "rwoption2",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption3": {
                    "name": "rwoption3",
                    "type": "float",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption4": {
                    "name": "rwoption4",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption5": {
                    "name": "rwoption5",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption6": {
                    "name": "rwoption6",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "rwoption7": {
                    "name": "rwoption7",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "1",
                    "max": "42",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "testkey": {
                    "name": "testkey",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "testlkey": {
                    "name": "testlkey",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "testnewline": {
                    "name": "testnewline",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "smb",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "internal_store_backend": {
                    "name": "internal_store_backend",
                    "type": "str",
                    "level": "dev",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "set internal store backend. for develoment and testing only",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "update_orchestration": {
                    "name": "update_orchestration",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "True",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "automatically update orchestration when smb resources are changed",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "snap_schedule",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "allow_m_granularity": {
                    "name": "allow_m_granularity",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "allow minute scheduled snapshots",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "dump_on_update": {
                    "name": "dump_on_update",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "dump database to debug log on update",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "stats",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "telegraf",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "address": {
                    "name": "address",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "unixgram:///tmp/telegraf.sock",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "interval": {
                    "name": "interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "15",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "test_orchestrator",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        },
        {
            "name": "zabbix",
            "can_run": true,
            "error_string": "",
            "module_options": {
                "discovery_interval": {
                    "name": "discovery_interval",
                    "type": "uint",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "100",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "identifier": {
                    "name": "identifier",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "interval": {
                    "name": "interval",
                    "type": "secs",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "60",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_level": {
                    "name": "log_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster": {
                    "name": "log_to_cluster",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_cluster_level": {
                    "name": "log_to_cluster_level",
                    "type": "str",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "info",
                    "min": "",
                    "max": "",
                    "enum_allowed": [
                        "",
                        "critical",
                        "debug",
                        "error",
                        "info",
                        "warning"
                    ],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "log_to_file": {
                    "name": "log_to_file",
                    "type": "bool",
                    "level": "advanced",
                    "flags": 1,
                    "default_value": "False",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "sqlite3_killpoint": {
                    "name": "sqlite3_killpoint",
                    "type": "int",
                    "level": "dev",
                    "flags": 1,
                    "default_value": "0",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "zabbix_host": {
                    "name": "zabbix_host",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "zabbix_port": {
                    "name": "zabbix_port",
                    "type": "int",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "10051",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                },
                "zabbix_sender": {
                    "name": "zabbix_sender",
                    "type": "str",
                    "level": "advanced",
                    "flags": 0,
                    "default_value": "/usr/bin/zabbix_sender",
                    "min": "",
                    "max": "",
                    "enum_allowed": [],
                    "desc": "",
                    "long_desc": "",
                    "tags": [],
                    "see_also": []
                }
            }
        }
    ]
}

</details>
