** **
# A CI for Development of UKL
** **
Project Members:  
- John Hunt | whunt@bu.edu  
- Zhassulan Kaishentayev jassulan@bu.edu  
- Camden Kronhaus | kronhaus@bu.edu  
- Samantha Puterman | samanpg@bu.edu  

## 1.   Vision and Goals Of The Project:

The UKL CI will provide a system to support continuous integration for the development of the linux unikernel (UKL) open source project. High level goals of this project include:  
* Providing a system of GitHub Actions to automatically compile the UKL, glibc and dependencies, and run test programs after any pushes and pull requests to the UKL repository to identify any bugs in the new features. 
* Ensuring the compiled kernel is able to boot and run on QEMU and produces the expected output.

## 2. Users/Personas Of The Project:
** **
The UKL CI will be used by all contributors to the open source UKL project who contribute to the <ins>UKL’s GitHub repository</ins> (https://github.com/unikernelLinux) via either a push or a pull request.

Notably, it does not target:
* Developers of non-UKL related projects

## 3.   Scope and Features Of The Project:

** **
The UKL CI will consist primarily of a series of scripts which are automatically triggered by GitHub Actions whenever pushes or pull requests are initiated in the repository (specifically on the current development branch, ukl-5.14). These scripts will include: 
* A build script which:

    * Fetches all repositories required for the unikernel compilation (unikernelLinux/ukl, unikernelLinux/linux, unikernelLinux/Linux-Configs, unikernelLinux/min-initrd, unikernelLinux/glibc, gcc-mirror/gcc)
  * Builds gcc, glibc, and other dependencies
  * Compiles the UKL kernel
* A boot script to launch a QEMU instance with the compiled UKL as the kernel
* Automated shell scripts which will run tests to ensure that QEMU boots and subsequently run test programs in QEMU to compare the actual output of these programs with expected results. 

## 4. Solution Concept
Global Architecture Description
Below is a description of the major components of our overall UKL CI system.**
* GitHub Actions: Triggered by a push or pull request to launch a runner and run CI scripts
* Runner: Server (running Linux) responsible for executing build scripts to clone UKL and dependency repositories, launch Docker Image (to boot QEMU), and run test scripts
* Docker Image: Container used to launch QEMU instance running compiled UKL  
    * Note: This is something we need to explore in greater detail. Initial exploration seems to suggest that booting QEMU in GitHub actions is best done using a Docker image (https://github.com/docker/setup-qemu-action), but this is something we will need to work on in-depth with our mentors as we design test scripts and mechanisms to extract the output from QEMU

![architecture diagram](./images/architecture-diagram.png)
                  
                              Figure 1: Basic Overview of Architecture/Workflow

<br></br>
Figure 1 details the overall architecture and workflow of the UKL CI system. A push or pull request from a user on the unikernelLinux/linux repository (on branch 5.14) will trigger a GitHub action which will launch a Linux-based runner. This runner will subsequently clone all required repositories for building the UKL, install dependencies, and execute a Makefile to build the UKL as a bootable kernel image. This kernel image will then be used to boot a QEMU emulator. Upon booting, test scripts will be executed to check the output of the emulator (note: we are still working to define what tests and tools we will use, but have been provided with some initial pointers to tools such as qemu-sanity-check (https://people.redhat.com/~rjones/qemu-sanity-check/) for checking the ability of qemu to boot).  
<br></br>
Design Implications and Discussions
* Running all actions on GitHub Hosted Runners: Currently, we plan to execute all build/test scripts on runners (machines) provided by GitHub. The principal reason for this design decision is that this will allow us to build a self-contained CI system (ie, not relying on external machines outside of GitHub). GitHub offers 2000 free build minutes a month, so if additional time is needed beyond this, we would need to explore using self-hosted runners. Moreover, we are still working to get the UKL to build on Ubuntu (the only Linux distribution offered in GitHub-hosted runners). If we are unsuccessful, we will likely need to use self-hosted runners with a different Linux distribution. Making this change would require us to set-up a 3rd-party VM to handle the Build/Test scripts whenever the GitHub action is triggered and subsequently clean up all created files after the action completes. 
* Running QEMU in a Docker Image: This initial design is mainly based on our understanding of the mechanisms to boot QEMU in GitHub. However, this component may change as we (1) Discover new mechanisms to run QEMU in GitHub Actions and/or (2) Discover we need to migrate to self-hosted runners



** Note: Our initial conceptual architecture is based on our assumptions that we will be able to build the MVP version of our system by solely using the “runner” machines provisioned by GitHub. However, this relies on a number of assumptions, including:
* The ability to get the UKL (and all dependencies) to compile on an Ubuntu Linux distribution (which is the only Linux distribution provided in GitHub actions). This is a task we’ve been attempting to solve since beginning the project and has been complicated by specific features of the gcc compiler used by Ubuntu  (notably, its default Position Independent Code setting) which thus far has prevented us from successfully building libgcc. Should we ultimately fail here, we will need to explore either how to run a different Linux distribution (e.g. Fedora) on a GitHub provided runner OR use a self-hosted runner which is running a compatible Linux distribution for the build.

## 5. Acceptance criteria
Minimum acceptance criteria is a CI running in GitHub actions that will test patches pushed to the UKL repository (or initiated via pull requests) by building the UKL, booting QEMU, and running test programs to alert the user early if the patch breaks the code.

Stretch goals for this project include:
* Running all build/test scripts on runners hosted outside of GitHub (such as at the MOC). This will provide greater flexibility in what type of system that kernel is built on (GitHub is limited to Ubuntu for Linux) and may allow us to test the kernel in bare metal.
* Testing for performance regressions as patches are pushed to the UKL or introduced via pull request (note: we do not currently have a methodology for doing this, so this will involve additional research should we pursue this goal).

## 6.  Release Planning:
** **
Release #1: (Due 10/1?)  
Github Action to run a workflow that compiles the UKL repository, glibc, gcc and other dependencies whenever there is a push or a pull request to the UKL repository and return the workflow status to the user, which can be success if the patch pushed introduced no failures or a failure status if the patch caused the repository not to compile. 

Release #2:  
Launching a QEMU instance through a boot script with the compiled UKL as the kernel and testing that it boots correctly. Currently, the plan is to use the qemu-sanity-check tool (developed by Richard Jones). https://people.redhat.com/~rjones/qemu-sanity-check/

Release #3:   
Successfully create (or replicate) a battery of system tests to ensure that the UKL behaves as expected while running in QEMU. Develop shell scripts to compare the output of the system tests to the expected output results. If they are not as expected, the Github Action should fail. 

Release #4:   
After delivering MVP (Releases 1-3), implement additional stretch targets as prioritized by our mentors (eg, running on bare metal or implement performance regression tests).*

*In our conversations thus far with our mentor (Richard) he has emphasized the MVP as the most valuable feature. We’ve spoken about the other (stretch) goals, but plan to revisit these with him after delivering the MVP. Based on this conversation, we can subsequently implement whichever features will drive the most value for the UKL project in the time remaining. 

## Computing Resources we may need
At the moment, we are operating on free-tier virtual machines for testing purposes and our initial plan is to use the machines running in GitHub for production. However, if we need to switch to self-hosted runners, we will need access to virtual machines which can receive triggers from GitHub actions and subsequently run the all build and test scripts. 
## General Comments


