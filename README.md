# expect-runner
This repository contains an expect/TCL script that could help users with automation. The idea is simple, send the command that needs to execute, the expected output, timeout, number, and retries. Then, the script will run for you. If you are good with `regex`, the whole concept should be simple for you to grasp.  you can find more about this script at https://technekey.com/one-expect-script-for-all-your-automation/
Something like this:

### Syntax:

A set of 4 arguments makes a command-set.So, the total number of args must be divisible by 4. 

./runner.exp <cmd1> <expected_output1> <timeout1> <retries1>  <cmd2> <expected_output2> <timeout2> <retries2>


### For example:

In the below example, the script will first execute the "date" command; then it will check for a string "2023" in the output of the "date" command; the timeout for the date command is ten seconds; if the output of the "date" command does not contain "2023." then retry maximum of 3 times. 

Then proceed to execute "uname" command, and look for "Linux" in its output; the maximum wait time is 10 seconds for uname command, and the max retry is set to 1. 

```
  ./runner.exp date 2023 10 3 uname Linux 10 1. 
```

### Pros:
1. This script has minimum dependencies, should be able to run on almost any linux machine. 
2. This is easy to integrate to any other scripting languages, just write a wrapper to pass the arguments. 
  
### Here is the output of the execution. Note that some examples might be trimmed for brevity. 

```
 ./runner.exp date 2023 10 3 "uname" Linux 10 1
Mon Feb  6 09:30:06 PM CST 2023 Info: Running command: date with pattern: 2023, timeout: 10 and retry count: 2 remaining retries
Mon Feb  6 09:30:06 PM CST 2023 Info: Command's(date) output consist: 2023 and within 10
Mon Feb  6 09:30:06 PM CST 2023 Info: Running command: uname with pattern: Linux, timeout: 10 and retry count: 0 remaining retries
Mon Feb  6 09:30:06 PM CST 2023 Info: Command's(uname) output consist: Linux and within 10
Consolidated Output:
$ date
Mon Feb  6 09:30:06 PM CST 2023

$ uname
Linux
```

A little more advanced usecase:

Here, SSH to a remote server is done and subsequent commands were executed. (be careful if you are sending sensitive info)

```
 ./runner.exp "ssh technekey@localstack-node-1" "password:" 10 3  "technekey" "technekey@.*$" 5 3 "ls -lrt /tmp" ".+@.+$" 5 1
Mon Feb  6 09:36:06 PM CST 2023 Info: Running command: ssh technekey@localstack-node-1 with pattern: password:, timeout: 10 and retry count: 2 remaining retries
Mon Feb  6 09:36:06 PM CST 2023 Info: Command's(ssh technekey@localstack-node-1) output consist: password: and within 10
Mon Feb  6 09:36:06 PM CST 2023 Info: Running command: technekey with pattern: technekey@.*$, timeout: 5 and retry count: 2 remaining retries
Mon Feb  6 09:36:06 PM CST 2023 Info: Command's(technekey) output consist: technekey@.*$ and within 5
Mon Feb  6 09:36:06 PM CST 2023 Info: Running command: ls -lrt /tmp with pattern: .+@.+$, timeout: 5 and retry count: 0 remaining retries
Mon Feb  6 09:36:06 PM CST 2023 Info: Command's(ls -lrt /tmp) output consist: .+@.+$ and within 5
Consolidated Output:
$ ssh technekey@localstack-node-1
technekey@localstack-node-1's password:
technekey@localstack-node-1:~$ls -lrt /tmp
total 24
drwx------ 3 root      root      4096 Feb  6 15:23 systemd-private-4d56bc17b8404d76b530ad796309861e-systemd-timesyncd.service-INo0wf
drwx------ 3 root      root      4096 Feb  6 15:23 systemd-private-4d56bc17b8404d76b530ad796309861e-systemd-resolved.service-7eIBCd
drwx------ 3 root      root      4096 Feb  6 15:23 systemd-private-4d56bc17b8404d76b530ad796309861e-systemd-logind.service-mxJpHy
drwx------ 3 root      root      4096 Feb  6 15:23 systemd-private-4d56bc17b8404d76b530ad796309861e-ModemManager.service-w7ZdOR
drwx------ 3 root      root      4096 Feb  6 15:23 snap-private-tmp
drwx------ 3 root      root      4096 Feb  7 02:26 systemd-private-4d56bc17b8404d76b530ad796309861e-fwupd.service-LRZoCC
-rw-rw-r-- 1 technekey technekey    0 Feb  7 02:56 foo
technekey@localstack-node-1:~$
```

Another example showing, how the changes done in one command-set(set of 4 args) is available to the next command-set(set of 4 args). In the below example an environment variable set is available to next command.   

```
./runner.exp "ssh technekey@localstack-node-1" "password:" 10 3  "technekey" "technekey@.*$" 5 3 "export foo=1234" ".+@.+$" 5 1 "printenv foo" "technekey@.*$" 5 3
Mon Feb  6 09:38:44 PM CST 2023 Info: Running command: ssh technekey@localstack-node-1 with pattern: password:, timeout: 10 and retry count: 2 remaining retries
Mon Feb  6 09:38:44 PM CST 2023 Info: Command's(ssh technekey@localstack-node-1) output consist: password: and within 10
Mon Feb  6 09:38:44 PM CST 2023 Info: Running command: technekey with pattern: technekey@.*$, timeout: 5 and retry count: 2 remaining retries
Mon Feb  6 09:38:44 PM CST 2023 Info: Command's(technekey) output consist: technekey@.*$ and within 5
Mon Feb  6 09:38:44 PM CST 2023 Info: Running command: export foo=1234 with pattern: .+@.+$, timeout: 5 and retry count: 0 remaining retries
Mon Feb  6 09:38:44 PM CST 2023 Info: Command's(export foo=1234) output consist: .+@.+$ and within 5
Mon Feb  6 09:38:44 PM CST 2023 Info: Running command: printenv foo with pattern: technekey@.*$, timeout: 5 and retry count: 2 remaining retries
Mon Feb  6 09:38:44 PM CST 2023 Info: Command's(printenv foo) output consist: technekey@.*$ and within 5
Consolidated Output:
$ ssh technekey@localstack-node-1
technekey@localstack-node-1's password:
technekey@localstack-node-1:~$export foo=1234
technekey@localstack-node-1:~$printenv foo
1234
technekey@localstack-node-1:~$
```
  
### Dependencies:
1. expect must be installed on your system. 

### Notes:
1. To parse the output you can pipe the expect script with below command:
```
perl -0777 -lne '/(?s)Consolidated [oO]utput:(.*)/;printf "%s\n",$1'
```
2. You can hide the output of the script by the setting print_enabled to 0 inside the script)
  
3. You can also send the execution details to syslog by setting send_to_log to 1. 
