TAOS-CI
=======


# Introduction
The Continuous Integration (CI) system is to prevent a regression and to find bugs due to incorrect PRs.
PRs causing regressions will not be automatically merged. We are going to report the issue at regression test procedure.

- Test automation (both build and run)
- Preventing performance regression
- Finding bugs at a proper time

# Overall flow
The below diagram shows an overall flow of CI system.
```bash

 |<-- Jenkins: Server Diagnosis -->|
 |                                 |
                       |<--------------------------- Standalone: Automation Area ------------------->|
                       |                                                                             |
                       |                                                                             |
1) Issue --> 2) PR --> 3) Build  ----> 4) Run Test  --> 5) Regression Check --> 6) Review ----> 7) Merged --> 8) Release
     |          |          |                |                    |                     |                           |
 (developers)   |(CIbot)   |(CIbot)         |(CIbot)         (git blame)           (reviewers)    (reviewers)      |(SR:Submit Request)
                |          |                |                                                                      |
                |          |-- Audit Modules`-- Pre-flight                                                         |-- Platform Image
                |          `-- Unit testing                                                                        `-- DashBoard
                 `-- Format Modules

```

# Prepare CI Server
There are two alternatives to maintain your own CI server.
* Standalone CI server: Use ./standalone/ folder after installing Apache and PHP in case of a small & lightweight project.
* Jenkins CI server: Use ./jenkins/ folder after installing Jenkins software (https://jenkins.io/) in case of a large & scalable project.

CIbot is github webhook handle template for a github repository in order to control and maintain effectively issues and PRs that are submitted by lots of contributors.
The official ID is git.bot.sec@samsung.com. Note that administrator has to sign-in +3 times every month to avoid a situation that ID is closed by Samsung SDS.

### How to install standalone CI software
* Bash: sh-compatible command language interpreter that can be configured to be POSIX-conformant by default.
* PHP: a widely-used general-purpose scripting language can be embedded into HTML.
* Curl: tool to transfer data to a CI server using the supported protocol such as HTTP/HTTPS.

We assume that you are using Ubuntu 16.04 64bit distribution. You can easily install required packages with apt-get command.

```bash
$ sudo apt-get -y install bash php curl
$ sudo apt-get -y install apache2
$ sudo apt-get -y install php php-cgi libapache2-mod-php php-common php-pear php-mbstring
$ sudo systemctl restart apache2
```

### How to install Jenkins CI software
We assume that you are using Ubuntu 16.04 64bit distribution. You can easily install Jenkins package with apt-get command.

```bash
wget -q -O - http://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins
sudo echo 'jenkins ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

sudo service jenkins stop 
sudo service jenkins start 
sudo service jenkins restart
chromium-browser http://localhost:8080/
```

# Self assessment: how to build a package
You have to execute ***gbs build*** command as a self-assessment before submitting your PR.
```bash
# in case of x86 64bit architecture
$ time gbs build -A x86_64  --clean --include-all
# in case of ARM 64bit architecture
$ time gbs build -A aarch64 --clean --include-all
```

# How to apply TAOS-CI into your project
```bash
$ cd /var/www/html/
$ git clone https://github.sec.samsung.net/STAR/<your_prj_name>.git
$ git clone https://github.sec.samsung.net/STAR/TAOS-CI.git
$ cd <your_prj_name>
$ cp -arfp ../TAOS-CI/{.github|doc|ci|UnitTestCoverageAssessment| ./
$ vi ./ci/standalone/config/botenv.sh
  Modify configuration variables appropriately.
```
That's all. Enjoy TAOS-CI after setting-up webhook API.

# How to use a webhook API

```bash
$ chromium-browser https://github.sec.samsung.net/STAR/AuDri/settings
```

Press `Hooks & services` menu - Press `Add webhook` button - 
```bash
* Webhooks/ Add webhook
  - Payload URL:  http://***.mooo.com/cibot.taos or http://***.mooo.com:8080/
  - Content type: application/x-www-form-urlencoded
  - Secret: ******
  - Which events would you like to trigger this webhook?
    [ ] Just the push event.
    [ ] Send me everything.
    [x] Let me select individual events.
      [v] Issues
      [v] Issue comment
      [v] Pull request
  - [v] Active
We will deliver event details when this hook is triggered. 
```

As a final step, press `Add webhook` button. That's all. From now on, enjoy CI world for more collaborative and productive software development!!!

### How to add new module

* plugins-good: it is a set of plug-ins that we consider to have good quality code, correct functionality, our preferred license (Apache for the plug-in code).
* plugins-ugly: it is a set of plug-ins that are not up to par compared to the rest. They might be close to being good quality, but they are missing something - be it a good code review, some documentation, a set of tests, or aging test.
