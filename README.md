** **
# A CI for Development of UKL
** **
Project Members:  
- Wiley Hunt | whunt@bu.edu  
- Zhassulan Kaishentayev jassulan@bu.edu  
- Camden Kronhaus | kronhaus@bu.edu  
- Samantha Puterman | samanpg@bu.edu  

## 1.   Vision and Goals Of The Project:

The UKL CI will provide a system to support continuous integration for the development of the linux unikernel (UKL) open source project. High level goals of this project include:  
* Providing a system of GitHub Actions to automatically compile the UKL, glibc and dependencies, and run test programs after any pushes and pull requests to the UKL repository to identify any bugs in the new features. 
* Ensuring the compiled kernel is able to boot and run test applications on QEMU and the test applications produce the expected output.

## 2. Users/Personas Of The Project:
** **
The UKL CI will be used by all contributors to the open source UKL project who contribute to the [UKL’s GitHub repository](https://github.com/unikernelLinux/linux) via either a push or a pull request.

Notably, it does not target:
* Developers of non-UKL related projects

## 3.   Scope and Features Of The Project:

** **
The UKL CI will consist primarily of a series of workflow scripts (written in .yaml as this is the markup language used by GitHub t configure workflows within GitHub actions) which are automatically triggered by GitHub Actions whenever pushes or pull requests are initiated in the repository (specifically on the current development branch, ukl-5.14). The tests will also run nightly via a cron-based scheduler in GitHub actions. These scripts will include: 
* A reusable "Cloning" Script (which can be used by all unit tests) which fetches all repositories required for the unikernel compilation (unikernelLinux/ukl, unikernelLinux/linux, unikernelLinux/Linux-Configs, unikernelLinux/min-initrd, unikernelLinux/glibc, gcc-mirror/gcc) and installs tools needed to build the UKL/run it in QEMU on the machine.
* A "Unit Tests" Workflow which runs the following test programs in the UKL (inside QEMU). Note: each program runs in parallel as a separate "job" on its own "runner" (a virtual machine hosted by GitHub in Azure)
  - Boot Test: A simple program that ensures that the UKL can successfully boot in QEMU
  - LE Bench System Calls Test: A test program which ensures that several common system calls (read, write, etc.) can run successfully in the UKL
  - Multi-threaded Test: A test program that ensures that a multithreaded application can successfully run in the UKL.
  - Memcached Test: A unit test which runs memcached in the UKL and tests the ability to set/get key value pairs in this memcached server. This unit test not only tests that memcached can run in the UKL, but also tests UKL networking capabilities.

* A "Configurations Tests" Workflow which individually compiles the UKL with every combination of the following UKL-specific configuration flags (UKL_SAME_STACK, UKL_USE_IST_PF, UKL_USE_RET) and runs the LE Bench System Calls test. Again, each program runs in parallel as a separate "job" on its own "runner" (a virtual machine hosted by GitHub in Azure)

* A "Latency Test" Workflow which compiles a modified version of LE Bench (which reports the time required for each individual system call test in the program) with the UKL and runs it in QEMU. This test has been implemented as a POC and tested on the following types of machines: GitHub-hosted VM (in Azure), self-hosted VM (in the MOC), and a physical server in Cloud Lab. This test has been added to the main unikernelLinux/linux repository, but is currently deactivated (so it does not run automatically). The test can easily be activated should the project secure a server they wish to dedicate to this test.


## 4. Solution Concept
Global Architecture Description
Below is a description of the major components of our overall UKL CI system.**
* GitHub Actions: Triggered by a push or pull request (or via a cron-based schedule) to launch a runner and run CI scripts
* Runner: Server (running Linux) responsible for executing build scripts to clone UKL and dependency repositories, boot QEMU, and run test scripts. Note: currently, we are actually using multiple runners so that different tests can be run independently in parallel.

![architecture diagram](./images/architecture-diagram.png)
                  
                              Figure 1: Basic Overview of Architecture/Workflow

<br></br>
Figure 1 details the overall architecture and workflow of the UKL CI system. A push or pull request from a user on the unikernelLinux/linux repository (on branch 5.14) will trigger a GitHub workflow which will launch a Linux-based runner for each test in the CI system (note: our final implementation also triggers workflows nightly via a cron-based scheduler). Each runner will subsequently clone all required repositories for building the UKL, install dependencies, and execute a Makefile to build the UKL (and a pre-defined test application) as a bootable kernel image. This kernel image will then be used to boot a QEMU emulator. Test scripts will be executed to check the output of the emulator to ensure that the test program(s) complete successfully.
<br></br>
Design Implications and Discussions
* Running (most) actions on GitHub Hosted Runners: 
   * The majority of our unit tests on runners (machines) provided by GitHub. The principal reason for this design decision is that this will allow us to build a self-contained CI system (ie, not relying on external machines outside of GitHub). GitHub offers unlimited build minutes for public repos (although a job cannot run for more than 6 hours), which should be more than sufficient for our tests (currently, the kernel takes approximately 40 minutes to compile). However the machine memory is capped at 7GB RAM and 14GB SSD, so if additional memory is needed for future tests, we would need to migrate these test to run on self-hosted runners. Making this change would require us to set-up a 3rd-party VM to handle the Build/Test scripts whenever the GitHub action is triggered and subsequently clean up all created files after the action completes (note: we have implemented this functionality as a Proof of Concept for a "latency test" action which we have tested on both an MOC server and a physical machine in Cloud Lab). 
   * By using GitHub hosted runners, we cannot run QEMU with KVM. This makes QEMU much more slowly. While our mentors have defined this "slow performance" as acceptable for now, we will need to ensure that it does not become an issue later on when we add more tests/compile the UKL with different applications.
   * As would be expected, GitHub-hosted runners do not allow us to test the UKL on bare metal. This is a stretch goal for the project, but should we get to that task, we will need to switch to using the MOC or another source where bare-metal testing is possible.


Repositories:

* Currently, CI testing and development is being performed in our [fork of the UKL repo](https://github.com/whunt1965/linux)

* As releases are completed, they are added to the [main UKL repo](https://github.com/unikernelLinux/linux)

## 5. Acceptance criteria
Minimum acceptance criteria is a CI running in GitHub actions that will test patches pushed to the UKL repository (or initiated via pull requests) by building the UKL, booting QEMU, and running test programs to alert the user early if the patch breaks the code.

Stretch goals for this project include:
* Running build/test scripts on runners hosted outside of GitHub (such as at the MOC). This will provide greater flexibility in what type of system that kernel is built on (GitHub is limited to Ubuntu for Linux) and may allow us to test the kernel in bare metal.
* Testing for performance regressions as patches are pushed to the UKL or introduced via pull request.

## 6.  Release Planning:
** **
Our Taiga Board can be found [here](https://tree.taiga.io/project/anqianqi1-csec528-fall-21-a-ci-for-development-of-ukl/timeline)

**Release #1: Implemented in Main UKL Repo on 10/5**

Github Action to run a workflow that compiles the UKL repository, glibc, gcc and other dependencies whenever there is a push or a pull request to the UKL repository and return the workflow status to the user, which can be success if the patch pushed introduced no failures or a failure status if the patch caused the repository not to compile. 

Action can be found [here](https://github.com/unikernelLinux/linux/actions)

**Release #2: Implemented in Main UKL Repo on 11/2**

Launching a QEMU instance through a boot script with the compiled UKL as the kernel and testing that it boots correctly. Note: This task was completed as one of the initial "unit tests" created for release 3. The action is available [here](https://github.com/whunt1965/linux/blob/ukl-5.14/.github/workflows/UKL_UNIT_TESTS.yml) and output of the action may be viewed [here](https://github.com/whunt1965/linux/actions/workflows/UKL_UNIT_TESTS.yml). Our plan is to release this feature along with other initial unit tests in Release 3.

**Release #3: Implemented in Main UKL Repo on 11/2**

Implement GitHub Action Workflows to run an initial set of Unit Tests. Each Action will individually compile the UKL with a test program, boot the UKL in QEMU, and check the output of QEMU (by streaming all QEMU output to a file which can be analyzed with a Bash script when QEMU shuts down) to ensure that the test program executed successfully. Our initial set of unit tests includes: 
* A test which simply ensures that the UKL boots (by checking the QEMU output for a string printed in the test program when the UKL boots)
* A test which runs a few simple system calls (based on the lebench test) and ensures that each part of the program completes successfully (by Checking the QEMU output for string printed after each of the system call tests complete)
* A test which compiles the UKL with each permutation of the specialized UKL configurations (developed by the UKL team) and ensures that the UKL can boot/run the above system calls test program in QEMU. 

These tests were first run (and evaluated) in [our fork](https://github.com/whunt1965/linux) of the UKL repo and were added to the main UKL repo on repository on 11/2. The actions can be viewed [here](https://github.com/unikernelLinux/linux/actions)

**Release #4: Implemented in Main UKL Repo on 11/20**

Implement the following additional unit tests to the workflow: 
* A test program to test the networking capabilities of the UKL. This test runs memcached in the UKL (inside QEMU) and tests whether the "host" runner (the machine in which the workflow is running) can read and write to memcached. 
* A test program to ensure UKL applications can run multiple threads. This test runs an application which spawns multiple threads (which each perform a simple tasks) and joins them. We then check the output of QEMU to make sure that the test ran successfully. 
* A test program which runs various system calls (based on the lebench test) and measures the time each system call test takes to complete. We then extract these times from the output of QEMU and report them in the action (and terminate the action to report an error if not all tests run). This test is a proof of concept for how latencies may be measured via GitHub actions and was tested on the following different typrs of machines: Github-hosted runner (VM in Azure), self-hosted runner (VM) in MOC, and self-hosted physical server in Cloud Lab. This test was added to the main UKL/linux repository but has been "deactivated" for now as the project does not have a server to dedicate to this test.

These tests were first run (and evaluated) in [our forked repository](https://github.com/whunt1965/linux/actions) and have subsequently been implemented in the main unikernelLinux/linux repository [here](https://github.com/unikernelLinux/linux/actions)

**Release #5: All Deliverables to Mentors by 12/8**

We have now validated and implemented our CI system into the [main UKL repository](https://github.com/unikernelLinux/linux/actions). For the final release, we are preparing the following for our mentors:
* A simple build script to test compiling/running an application in the UKL (locally):
  -  This script has been completed and added to a new repository within the project [here](https://github.com/unikernelLinux/ukl-build)
* Detailed Documentation on our CI System: 
  - This document has been prepared and is available [here](https://drive.google.com/drive/u/0/folders/1QYZJbnxKFFk1BDBsm6t5EY2wLc1FAJ5e)
* Tutorial on GitHub Actions
  - We have prepared a (short) document providing an overview of GitHub actions and some of the lessons we learned through our project. This document will be provided to both our mentors and the course staff so that future projects may make use of it. 
  - The document is available [here](https://drive.google.com/drive/u/0/folders/1QYZJbnxKFFk1BDBsm6t5EY2wLc1FAJ5e)

** **
## Sprint Demo Videos

Sprint 1: https://drive.google.com/file/d/1uyXb03ig_F_Q8udL5lgs5sNOPHZjnRgZ/view?usp=sharing

Sprint 2: https://drive.google.com/file/d/159fcWlAx-YyuqLVU3G0UOF5Mf6WIg77E/view?usp=sharing

Sprint 3: https://drive.google.com/file/d/1rxiRq5U2SMQPrR4_HnuqhIxqB6eEjs84/view?usp=sharing

Sprint 4: https://drive.google.com/file/d/1D601Hhs6gQDGnI2LJIvR5eQfntei2Qeu/view?usp=sharing

Sprint 5: https://drive.google.com/file/d/1MqF_6at1HVH_peVMaq6H9_b3S3NzCuok/view?usp=sharing
