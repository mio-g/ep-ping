# Job and cron jobs

## Kubernetes Job / Batch Job

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate.
As pods successfully complete, the Job tracks the successful completions.

When a specified number of successful completions is reached, the task (ie, Job) is complete.  Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

A simple case is to create one Job object in order to reliably run one Pod to completion.
The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

You can also use a Job to run multiple Pods in parallel.

### Example

This example CronJob manifest prints the current time and a hello message every minute:


```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: countdouwn-with-timeout
spec:
  backoffLimit: 5
  activeDeadlineSeconds: 100
  template:
    spec:
      containers:
      - name: counter
        image: centos:7
        command:
         - "bin/bash"
         - "-c"
         - "for i in 5 4 3 2 1 ; do echo $i ; done"
      restartPolicy: Never
```

If you want to run a Job (either a single task, or several in parallel) on a schedule,
see **CronJob** below.

## Kubernetes CronJob

CronJobs are meant for performing regular scheduled actions such as backups,
report generation, and so on. Each of those tasks should be configured to recur
indefinitely (for example: once a day / week / month); you can define the point
in time within that interval when the job should start.

### Example

This example CronJob manifest prints the current time and a hello message every minute:

```yaml

apiVersion: batch/v1
kind: CronJob
metadata:
  name: check-kernel
spec:
  jobTemplate:
    metadata:
      name: check-time
    spec:
      template:
        spec:
          containers:
          - image: alpine:latest
            name: check-time
            command:
            - "/bin/sh"
            - "-c"
            - "uname -a"
            resources: {}
          restartPolicy: OnFailure
  schedule: '* * * * *'

```

([Running Automated Tasks with a CronJob](/docs/tasks/job/automated-tasks-with-cron-jobs/)
takes you through this example in more detail).

### Cron schedule syntax

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
```

| Entry 										| Description																									| Equivalent to |
| ------------- 						| ------------- 																							|-------------  |
| @yearly (or @annually)		| Run once a year at midnight of 1 January										| 0 0 1 1 * 		|
| @monthly 									| Run once a month at midnight of the first day of the month	| 0 0 1 * * 		|
| @weekly 									| Run once a week at midnight on Sunday morning								| 0 0 * * 0 		|
| @daily (or @midnight)			| Run once a day at midnight																	| 0 0 * * * 		|
| @hourly 									| Run once an hour at the beginning of the hour								| 0 * * * * 		|



For example, the line below states that the task must be started every Friday at midnight, as well as on the 13th of each month at midnight:

`0 0 13 * 5`

To generate CronJob schedule expressions, you can also use web tools like [crontab.guru](https://crontab.guru/).

---

🆙 next - [Summary](04-summary.md) 