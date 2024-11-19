# Processes in InstructLab

The ability to detach from processes is crucial to the user experience of InstructLab. However, the concept of multi-processing, process management, and the monitoring of processes is very complex.

It is important to try and add this concept in as simply as possible, expanding on the state reporting, logging, and other features as we go along.

## Phased approach to InstructLab Processes

This document is going to describe phase 1 of implementing processes in InstructLab. Phase 1 is to be described as the "ilab simple process management system". This will depend purely on python packages, PID tracking, and log files to create the experience of detachable processes. The key here is the concept of the UUID, allowing a future REST API to keep track of InstructLab processes using these unique identifiers.

We can re-visit all this in phase 2, when we discuss if we want to utilize something like systemd or a more in-depth process-monitor repo to track processes.

### Phase 1

Phase one would focus on adding the concept of detaching from processes. This would be implemented in `ilab 0.23.0`.

Process management would only apply to `ilab data generate` and `ilab model train` in a first iteration. This would be followed by commands like `ilab model evaluate`, `ilab model serve`, and `ilab model download`. All of these commands have long running processes that would benefit from detachment.

The workflow would allow for:

`ilab data generate -dt` (run a detached generation process)
`ilab model train -dt` (run a detached training process)

`ilab process list`

```console=
+----------------------------------------+------------+-------------+-------------+---------+
| Proccess UUID                          | PID        |  Type       | Duration    | Status  |
+----------------------------------------+--------------------------+-------------+---------+
| train-2423813fhthfbnaj                 |  33451     |  training   | 8 hr 32 min | Running |
| generation-6768769599sdbjklm           |  35543     |  generation | 3 hr 15 min | Running |
| generation-1345661hlsmma               |  45424     |  generation | 22 min      | Errored |
+----------------------------------------+---------------------+------------------+---------+

```

`ilab process attach <UUID>`

This command would re-attach to the given process, allowing to user to view the live logs of the process. `attach` would trail the log file and listen for user-input to kill the process.

These commands will be done in a very simple way at first using the following architecture:

2. a detached process be re-attachable by tailing the log file and then allowing the user to ctrl+c the process as normal using `KeyboardInterrupt`
3. The process registry will be maintained for tracking UUIDs created via the `uuid` python package, the PID of the actual process, a `log_file` where the process will be outputting its logs to so that the user can re-attach, and the start time of the process. The log file directory will be tracked using our `DEFAULTS` package and will be standard throughout releases.

The general flow would be:

1. a user runs `ilab data generate -dt`
2. a UUID, PID, and log file is added to the process registry.
3. the process would exit, and print the UUID of the sdg run
4. a user could attach to this process using `ilab process attach <UUID>`.
5. This command would look in the process registry for the PID and/or UUID, get the log file, tail the log file, and listen for a ctrl+c keyboard interrupt.

This allows us to detach from processes while still running them in the background and maintain log files all without the use of anything other than UUID and subprocess.

#### log file management

If existing log files from the various libraries exist, those will be used in this scenario. If they do not, InstructLab will manage writing process logs to disk. Regardless of whether the libraries maintain their own log file, InstructLab will need to co-locate the log files in a centralized directory.

If a log file exists, it will be copied and renamed into the following directory format:

`~/.local/share/instructlab/logs/<command_name>/<command_name>-<timestamp>.log`

If the log file does not exist, InstructLab will create one with this format. Libraries are responsible for standardizing where their logs are stored if they already exist so the CLI can access them in a uniform fashion and copy them to the proper directory.