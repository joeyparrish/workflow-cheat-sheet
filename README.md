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

### Common fields

|         |github.event_name  |github.ref        |
|---------|-------------------|------------------|
|Push     |push               |refs/heads/main   |
|PR       |pull_request       |refs/pull/33/merge|
|PR Target|pull_request_target|refs/heads/main   |
|Release  |release            |refs/tags/v1.0.0  |
|Schedule |schedule           |refs/heads/main   |
|Dispatch |workflow_dispatch  |refs/heads/main   |
|Call     |(from parent)      |(from parent)     |

Any `workflow_call` triggers inherit the entire event from the parent workflow.
If your workflow can be called from multiple events, it is probably best to
rely on explicit inputs.


### Specific fields for push

|         |github.event_name  |github.ref        |github.event.after|
|---------|-------------------|------------------|------------------|
|Push     |push               |refs/heads/main   |e62294f...        |

`github.ref` has the branch, but `github.event.after` has the exact SHA1.  If
you test each push without canceling the workflow when new pushes happen, use
the SHA1.  If you only need to test the latest in the branch and cancel any
pending workflow runs from older pushes, you can use the branch.


### Specific fields for release

|         |github.event_name  |github.ref        |github.event.release.tag_name|
|---------|-------------------|------------------|-----------------------------|
|Release  |release            |refs/heads/v1.0.0 |v1.0.0                       |

`github.ref` points to the exact release ref, but if you need the tag name,
don't parse it.  Just use `github.event.release.tag_name`.


### Specific fields for PRs

|         |github.event_name  |github.ref        |github.event.number|github.event.head.sha|
|---------|-------------------|------------------|-------------------|---------------------|
|PR       |pull_request       |refs/pull/33/merge|33                 |82db69e...           |
|PR Target|pull_request_target|refs/heads/main   |33                 |82db69e...           |

`github.ref` is only meaningful for `pull_request` triggers, but not
`pull_request_target` triggers.  So beware when transitioning from one to the
other.  You can use `github.event.head.sha` in both cases to get a specific
universal ref that works for both.  This is the exact pushed SHA1, not a merge
of that into the base.  You can also get the PR number from
`github.event.number`, or even construct a ref like
`refs/pull/${{github.event.number}}/merge` to get a universal merge ref.


## Basic Checkout Patterns

You should probably base your actions/checkout ref on one of these, or an
ordered combination of them when multiple triggers are used.

 * ~Push (branch, branch name): `${{ github.ref }}`~
 * Push (branch, specific SHA1): `${{ github.event.after }}`
 * ~PR (symbolic): `${{ github.ref }}`~
 * PR / PR Target (specific SHA1): `${{ github.event.head.sha }}`
 * PR / PR Target (merge commit): `refs/pull/${{ github.event.number }}/merge`
 * Dispatch (manual, ref): `${{ inputs.ref }}`
 * Dispatch (manual, PR): `${{ format('refs/pull/{0}/head', inputs.pr) }}`
 * Call (passed from another workflow): `${{ inputs.ref }}`
 * Release: `${{ github.ref }}`
 * Schedule: default branch name


## Basic Concurrency Group Patterns

You should probably set your concurrency group based on one of these, or an
ordered combination of them when multiple triggers are used.

 * Push (branch): `${{ github.workflow }}-${{ github.ref }}`
 * PR / PR Target: `${{ github.workflow }}-${{ github.event.number }}`
 * Release: `${{ github.workflow }}-${{ github.ref }}`
 * Schedule: `${{ github.workflow }}`
 * Dispatch: based on inputs
 * Call: based on inputs
