# Start building an application: draw a BPMN process diagram

Now that you have your development environment (i.e. Bonita Studio) up and running, you are ready to start building your first Bonita Living Application.

In this tutorial you will build a **claims management solution**. Making sure that claims are handled and answered in a timely manner by the appropriate employee(s) can be key to customer satisfaction. It is also a good fit for a process-based application that uses automation. We will of course drastically simplify the application in this tutorial so you can focus on learning about Bonita and not about claims management!

Here is an overview of process execution:

- The process will be started by a *customer* who submits a claim, which is a simple text description in our example
- Next, an *employee* will review the claim and provide an answer
- The *customer* will be able to review the *employee*'s answer and provide a satisfaction rating
- If the rating is equal to or greater than 3, the process will finish after the customer has read the answer
- If the rating is lower than 3 an extra task will be assigned to the manager of the employee to work with this customer who is not really satisfied. The manager will be notified by email that a task Deal with unsatisfied customer requires their attention.
- Once this task is done the process will be completed


::: info
To design this process we will use the Business Process Model and Notation (BPMN) standard. You don't need to learn everything about BPMN to follow this tutorial but, if you would like to know more, you can read the [Ultimate Guide to BPMN 2.0](https://www.bonitasoft.com/library/ultimate-guide-bpmn).
:::

::: warning
We will use an iterative approach in the process development. This means that in the first iteration, we will have a few limitations:

- a single user will perform all the tasks
- we won't collect data from the user
- the process will follow a single path - in this case the one for an unsatisfied customer - which will be called "happy path" because its design is the best case scenario to run the entire process smoothly.

As we introduce data in our process definition and configure actors, a user will be able to input data in web forms, the process will follow paths that depend on data values, and different users will be involved.
:::

## Create the process diagram

The first step is the design of the process "happy path."

::: info
While drawing the model, you will use and learn about some basic BPMN elements used in most process diagrams: 
- [Pool](pools-and-lanes.md): a pool is a container for a process in a diagram.
- [Lanes](pools-and-lanes.md): a lane is a division of a pool. Typically, a lane contains all the tasks assigned to one [actor](actors).
- [Start and end events](events.md): a start event is at the start of a process flow, and an end event is at the end of a flow.
- [Tasks](diagram-tasks.md): a task is a type of activity in a process.
- [Gateways](gateways.md): gateways are used to control how sequence flows interact within a process.
:::


Create a new diagram:

1. In the Bonita Studio menu, click on **File > New diagram**.

   ![Add user task](images/getting-started-tutorial/draw-bpmn-diagram/new-diagram.gif)<!--{.img-responsive .img-thumbnail}-->

The new diagram includes:

- One pool: one process definition
- One lane inside the pool: lane default name is _Employee lane_ (more about lanes in another chapter)
- A start event: the starting point of process execution
- A user task: a task (or step) that needs to be performed by a user

and it should look like this:

![Default diagram content](images/getting-started-tutorial/draw-bpmn-diagram/new-default-diagram.png)

Rename the diagram and the pool:

1. In the **Project explorer** on the left-hand side of the Bonita Studio window, select the diagram _MyDiagram-1.0.proc_
2. Right click and select **Rename...**
   ![Rename contextual menu](images/getting-started-tutorial/draw-bpmn-diagram/rename.png)
3. In the pop-up window, enter the new name for the diagram: _ClaimsManagementDiagram_, and for the pool: _ClaimsManagement_
   ![Choose a new name and version pop-up window](images/getting-started-tutorial/draw-bpmn-diagram/choose-new-name-version.png)

Rename the start event (i.e. the green disc on the left hand side of the process):
1. Select the start event in the diagram
2. At the bottom of the Bonita Studio screen, go to the **General > General** tab
3. Edit the **Name** attribute and set the value: _Submit claim_

   ![Rename start event](images/getting-started-tutorial/draw-bpmn-diagram/rename-start-event.png)

Also rename the default user task:

1. Select the user task _Step1_
2. At the bottom of the Bonita Studio screen go to the **General > General** tab
3. Edit the **Name** attribute and set the value: _Review and answer claim_

Add another task to the process:
1. Select the _Review and answer claim_ task
2. At the border of the selected task, click on the task icon and drag and drop it onto the diagram
3. Make sure that the newly added task is selected
4. Click on the toolbox icon
5. Select the user icon
6. Rename the task to: _Read the answer and rate it_

   ![Add user task](images/getting-started-tutorial/draw-bpmn-diagram/add-task.gif)

In order to define another path in the process execution, add an exclusive gateway. This gateway will direct the process execution to a different path based on certain conditions. Adding an exclusive gateway is done in the same way as adding a task:
1. Select the user task _Read the answer and rate it_
2. Drag and drop the diamond-shaped gateway icon
3. Click on the toolbox icon
4. Change the gateway type from parallel (the one with the **+** sign) to exclusive (the one with the **x** sign)
   
   ![Switch from parallel to exclusive gateway](images/getting-started-tutorial/draw-bpmn-diagram/switch-from-parallel-to-exclusive-gateway.png)
   
5. Rename the gateway to: _Satisfaction level_

From the gateway, there are two possibles paths. The first one completes the process if the customer satisfaction level is good. The end of a process is identified by an end event, a red disc. Add the end event in a similar way as adding a task and a gateway:
1. Select the gateway
2. Drag and drop the event icon (the disc icon)
3. Select the end event type (red disc icon)

   ![Event type selection: end event](images/getting-started-tutorial/draw-bpmn-diagram/event-type-selection.png)

4. Rename the event to: _End client satisfied_

The second path from the gateway will lead to another user task:

1. Select the _Satisfaction level_ gateway
2. Drag and drop the task icon
3. Click on the toolbox icon
4. Select the user icon
5. Rename the task _Deal with unsatisfied customer_

And from this _Deal with unsatisfied customer_ task the path will go to a second end event:
1. Select the _Deal with unsatisfied customer_ user task
2. Drag and drop the event icon (the disc icon)
3. Select the end event type (red disc icon)
4. Rename the event to: _End client unsatisfied_
1. Refresh the diagram validation: at the bottom of the screen, select the **Validation status** tab and click on **Refresh**

Your process diagram should look like this:

![Process diagram](images/getting-started-tutorial/draw-bpmn-diagram/process-diagram-before-transitions-configuration.png)

::: info
Now you can see a validation error on the gateway. The error is displayed because we didn't define the conditions for process execution when it reaches the gateway. Trying to execute the process will fail because the process execution engine cannot determine which path to follow.
:::

Configure the conditions required to define the path to follow when executing the process. Conditions are defined on the gateway outgoing transitions (i.e. the arrows going out of the gateway):
1. Select the transition going to the end event
2. Name it to _Good_
3. Go to the **General > General** tab
4. Check the checkbox **Default flow**

   ![Transitions names and conditions configuration](images/getting-started-tutorial/draw-bpmn-diagram/transitions-name-and-condition.gif)

::: info
**Default flow** means that the transition will be activated only if all other conditions configured on the gateway outgoing transitions are _false_ (boolean value).
:::

Configure the condition on the transition from the gateway to the _Deal with unsatisfied customer_ task:
1. Select the transition going into the task _Deal with unsatisfied customer_
2. Name the transition _Bad_
3. Go in **General > General** tab
4. In **Condition** type the value _true_
1. In the Bonita Studio menu, click on **File > Save**

::: warning
Due to the static value _true_, this version of the process will always go through the _Deal with unsatisfied customer_ task.
:::

::: info
To make the error go away, refresh the diagram validation. At the bottom of the screen, select the **Validation status** tab and click on **Refresh**. You should have only five warnings left.
:::

Congratulations! You have now your first valid BPMN process diagram!

## Run the process

You can now build, package, deploy and execute this process definition in the Bonita Studio embedded test environment:
1. Select the process pool, the rectangle shape that includes start events and tasks
2. Click on **Run** ![Run button](images/getting-started-tutorial/draw-bpmn-diagram/run.png) in the toolbar at the top of the Bonita Studio window
3. On the instantiation form, click on **Start**. The process instance is started and you are redirected to the Bonita Portal task list
4. Move your mouse over the _Review and answer claim_ form on the right hand side of the Bonita Portal and click on **Take** to "claim" the task
5. Click on the **Execute** button to actually perform the task and move the process execution forward
6. Click on the **Refresh** button ![Refresh button](images/getting-started-tutorial/draw-bpmn-diagram/refresh.png) at the top of the task list to update it
7. Click on the **Overview** tab on the right side to see the process execution overview form with information about process execution
8. Click on the **Form** tab to get back to the _Read the answer and rate it_ user task form
9. You can click on **Take** and execute for both tasks, _Read the answer and rate it_ and _Deal with unsatisfied customer_, to finish the process execution
10. In the Bonita Portal top menu, click on **Cases** ![Cases button](images/getting-started-tutorial/draw-bpmn-diagram/cases.png)
11. Click on the **Archived cases** tab
12. Click on the **View case overview** action (i.e. the "eye" icon) to display the overview form with information about process execution
   
   ![Process execution](images/getting-started-tutorial/draw-bpmn-diagram/run-process.gif)<!--{.img-responsive .img-thumbnail}-->

::: info
When you click on the **Run** button, the process definition and its dependencies are built, packaged and deployed in the Bonita Studio test environment. A user is logged in by default (username: _walter.bates_, password: _bpm_) and the auto-generated start form for the process is opened in your web browser. If you submit the instantiation form, it will start a new process instance (or case) and load the user task list in the Bonita Portal. In the task list, you can't immediately submit a user task because, by default, all users (of the test organization) are candidates to perform the tasks of the process. In order to act on the task you need first to "claim" it, which then makes you the only one - among all the possible candidates - who can do perform an action on it.
 :::

::: info
You can view process instance information in the [**Cases**](cases.md) section of the Bonita Portal. Switch between **Open cases** and **Archived cases** to view the ongoing process instances or see completed ones. Note that if you want to start a second case (i.e. a process instance), you must go into the Bonita Portal **Processes** menu and click on the **Start a new case** button (i.e. the "play" icon in the **Action** column) next to the process definition name. If you click on **Run** from Bonita Studio, it will clean / overwrite any information related to any process with same name and version, including previous cases. Note that if you did any modifications to your project, you probably want to click on **Run** to be sure that the latest version is deployed.
:::

You've successfully built and executed your first process. So far it is not very useful as it does not deal with any data and web forms, and  only includes a single submit button. In following chapters you will [define a business data model](define-business-data-model.md) and bind it to the process to save information related to the claim submitted by the user.