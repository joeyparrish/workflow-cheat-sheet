# GitHub Workflow Cheat Sheet

It's so easy to make typos in a GitHub Actions variable, and you get no
feedback on it.  Your workflow just does the wrong thing.

Even if you use a real variable name, it's easy to use the wrong one.  You
could check out the wrong revision, or fail to cancel an expensive workflow, or
cancel the wrong instance of a workflow.

Here are some sample values of various variables available to your workflow,
and recommended patterns for actions/checkout and for concurrency groups for
each common trigger.


## Sample Event Info Per Trigger

|         |github.event_name  |github.ref        |
|---------|-------------------|------------------|
|Push     |push               |refs/heads/main   |
|PR       |pull_request       |refs/pull/33/merge|
|PR Target|pull_request_target|refs/heads/main   |
|Release  |release            |refs/tags/v1.0.0  |
|Schedule |schedule           |refs/heads/main   |
|Dispatch |workflow_dispatch  |refs/heads/main   |
|Call     |(from parent)      |(from parent)     |


|         |github.event_name  |github.ref        |github.event.after|
|---------|-------------------|------------------|------------------|
|Push     |push               |refs/heads/main   |e62294f...        |


|         |github.event_name  |github.ref        |github.event.release.tag_name|
|---------|-------------------|------------------|-----------------------------|
|Release  |release            |refs/heads/v1.0.0 |v1.0.0                       |


|         |github.event_name  |github.ref        |github.event.number|github.event.head.sha|
|---------|-------------------|------------------|-------------------|---------------------|
|PR       |pull_request       |refs/pull/33/merge|33                 |82db69e...           |
|PR Target|pull_request_target|refs/heads/main   |33                 |82db69e...           |


## Basic Checkout Patterns

 * ~Push (branch, branch name): github.ref~
 * Push (branch, specific SHA1): github.event.after
 * ~PR (symbolic): github.ref~
 * PR (specific SHA1): github.event.head.sha
 * PR Target (specific SHA1 required): github.event.head.sha
 * Dispatch (manual, ref): inputs.ref
 * Dispatch (manual, PR): format('refs/pull/{0}/head', inputs.pr)
 * Call (passed from another workflow): inputs.ref
 * Release: github.ref
 * Schedule: default branch name


## Basic Concurrency Group Patterns

 * Push (branch): ${{ github.workflow }}-${{ github.ref }}
 * PR / PR Target: ${{ github.workflow }}-${{ github.event.number }}
 * Release: ${{ github.workflow }}-${{ 
 * Schedule: ${{ github.workflow }}
 * Dispatch: based on inputs
 * Call: based on inputs
