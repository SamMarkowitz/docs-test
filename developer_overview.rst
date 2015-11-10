Developer Overview
++++++++++++++++++

What follows is a brief overview of how CloudSlang and the CloudSlang
Orchestration Engine (Score) work. For more detailed information see the
`Score API <developer_score.md#score-api>`__ and `Slang
API <developer_cloudslang.md#slang-api>`__ sections.

The CloudSlang Orchestration Engine is an engine that runs workflows.
Internally, the workflows are represented as
`ExecutionPlans <developer_score.md#executionplan>`__. An
`ExecutionPlan <developer_score.md#executionplan>`__ is essentially a
map of IDs and `ExecutionSteps <developer_score.md#executionstep>`__.
Each `ExecutionStep <developer_score.md#executionstep>`__ contains
information for calling an action method and a navigation method.

When an `ExecutionPlan <developer_score.md#executionplan>`__ is
triggered it executes the first
`ExecutionStep's <developer_score.md#executionstep>`__ action method and
navigation method. The navigation method returns the ID of the next
`ExecutionStep <developer_score.md#executionstep>`__ to run. Execution
continues in this manner, successively calling the next
`ExecutionStep's <developer_score.md#executionstep>`__ action and
navigation methods, until a navigation method returns ``null`` to
indicate the end of the flow.

CloudSlang plugs into the CloudSlang Orchestration Engine (Score) by
compiling its workflow and operation files into Score
`ExecutionPlans <developer_score.md#executionplan>`__ and then
triggering them. Generally, when working with CloudSlang content, all
interaction with Score goes through the `Slang
API <developer_cloudslang.md#slang-api>`__, not the `Score
API <developer_score.md#score-api>`__.
