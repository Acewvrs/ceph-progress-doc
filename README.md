# ceph-progress-doc

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
 
### 10/29
Document everything & progress we made

At this point, we're ready to commit our work! 

Read the PR guideline for Ceph, which is [here](https://github.com/ceph/ceph/blob/main/SubmittingPatches.rst). <br />
I ensured our commit titles and the content of PR were adhering to the guidelines. <br />
After everything looked good, I raised a [pull request](https://github.com/ceph/ceph/pull/60548)



