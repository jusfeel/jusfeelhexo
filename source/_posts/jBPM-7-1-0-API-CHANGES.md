---
title: jBPM 7.1.0 API CHANGES
date: 2018-10-28 21:58:46
tags:
---
> Saving time is saving lives.&nbsp;

If you were using jBPM6.x, the rest API endpoint is&nbsp;_"&lt;server&gt;/jbpm-console/rest/..."_.&nbsp;

New version 7.1.0, this won't work. The lengthy documentation does not mention this change obviously. Though, it is&nbsp;[there](https://docs.jboss.org/jbpm/release/7.1.0.Final/jbpm-docs/html_single/#_process_execution_server). And the real deal is&nbsp;[somewhere else](https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/development_guide/realtime_decision_server).

And I shall quote:
> [25.2.1.2. Process Execution Server](https://docs.jboss.org/jbpm/release/7.1.0.Final/jbpm-docs/html_single/#_process_execution_server)
> 
> The process execution server (also known as kie-server) has been extended to support the core engine features above (related to case management, admin APIs, etc.) and to offer a remote API for these operations. On top of that, two other imp<wbr>ortant architectural changes were done.
> 
> [](https://docs.jboss.org/jbpm/release/7.1.0.Final/jbpm-docs/html_single/#_separate_workbench_from_execution_server)[Separate workbench from execution server](https://docs.jboss.org/jbpm/release/7.1.0.Final/jbpm-docs/html_single/#_separate_workbench_from_execution_server)
> 
> While in v6 the workbench came with an embedded execution server to execute all the process and task requests that users were performing in the web-based UI, in v7 this embedded execution server has been removed and the workbench delegates all its requests to the kie-server as well. The main advantage is that the workbench can now be used to monitor any (set of) kie-server(s). By linking the kie-server to the workbench, the process and task monitoring UIs in the workbench can now connect to this kie-server and show all relevant information. When multiple independent kie-servers are used, you can either connect to a specific on<wbr>e or use the smart router to aggregate information across multiple servers (see below). As a result, a few missing features that were not yet available in v6 on kie-server but on<wbr>ly on the remote API of the workbench have also been migrated to the kie-server.

The real documentation for 7.1.0 can be found in 6.* version documentation in&nbsp;[here](https://access.redhat.com/documentation/en-us/red_hat_jboss_bpm_suite/6.4/html/development_guide/realtime_decision_server)

As I have asked about n SO, I think it's worth pointing out what the new endpoint is and where the documents about them are.

After you_&nbsp;build &amp; deploy&nbsp;_your project&nbsp; under&nbsp;_project authoring_, under deploy/execution servers, you can see the kie-server API endpoint for the container already.&nbsp;_h_

_
ttp://localhost:8080/kie-server/services/rest/server/containers/TrainingPortal_1.0.0_

_
_

You can now check the process definitions if you have created processes by visiting&nbsp;

_http://localhost:8080/kie-server/services/rest/server/containers/TrainingPortal_1.0.0/processes&nbsp;_

_
_

Started the process, you can see it in:

_http://localhost:8080/kie-server/services/rest/server/containers/TrainingPortal_1.0.0/processes/instances_
and,

_http://localhost:8080/kie-server/services/rest/server/containers/TrainingPortal_1.0.0/processes/instances/1_
(1 is the instance ID)