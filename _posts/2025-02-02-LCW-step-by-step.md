---
title: "Lifecycle Workflows and Custom Extensions - step-by-step-guide"
author: Klaus Bierschenk
date: 2025-02-02T08:00:32

layout: list

image:
  path: /MyPics/2025-02-02-LCW-Cover.jpg
---

<figure>
  <img src="/MyPics/2025-02-02-LCW-Cover.jpg" style="width:75%">
  <figcaption>Designed by vectorjuice / Freepik</figcaption>
</figure>


* this unordered seed list will be replaced by the toc
{:toc}

Welcome to my new blog post!

Lifecycle Workflows are a fantastic way to trigger reactions based on user information, consisting of multiple workflow steps. A classic example involves the two user attributes **“employeeHireDate”** and **“employeeLeaveDateTime”**. Based on the timestamps they contain, actions can be triggered either at an exact time or a certain period before or after the timestamp —this can be configured as needed.

Other triggers for workflows can include changes in group memberships or modifications to a user account, such as updates to the **"Department"** attribute or any other attribute.

In this [Article](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/how-to-configure-entra-to-active-directory) Microsoft describes this topic with a focus in the Identity Governance Dashboard. Interaction of Lifecycle Workflow and the Custom Extension Logic App is not part of this document.

And if you’re new to the subject and not very familiar with Lifecycle Workflows, especially Custom Extensions and Logic Apps, you may quickly run into challenges when trying to set this up in a lab environment. In addition to the Microsoft examples, there are many great examples that describe working with Custom Extensions. However, I’ve noticed that most of them assume prior knowledge of Logic Apps, which not everyone may have. Especially, the parameterization of actions and the interaction with User Identity Informations can be challenging at various points.

That’s why this article provides a step-by-step guide outlining the individual steps needed to add a Custom Extension and highlighting key aspects to watch out for.

For the lifecycle workflow steps, there are predefined tasks that cover the most common functions, such as assigning licenses, modifying group memberships, and more. However, administrators quickly reach the limits of the predefined options—so what then? This is exactly where Custom Extensions comes into the game. They consist of a Logic App, which in turn becomes a task within a workflow.

For our example, let’s say you want to send an email with relevant information to a new hire’s personal email address, welcoming them with some basic information. Since private email addresses are not standard attributes in a Lifecycle Workflow, we use the Graph API within the Logic App to retrieve all relevant details about the user object. Along the way, we’ll explore the actions that make up the Logic App and the potential they offer.

The following step-by-step guide consists of the following tasks:
* Creating the Custom Extension from the Lifecycle Workflow Dashboard with a Managed Identity
* Granting the Managed Identity the necessary permissions to execute the required actions
* Executing a Graph API query to retrieve user object information (e.g., otherMails)
* Sending the email to the address contained in otherMails via a dedicated action
* Returning an “OK” status to the Lifecycle Workflow

After this overview, we will now take a closer look at each step in detail.

The starting point is the Lifecycle Workflows dashboard in the Entra Admin Center.

[![LCW_1](/MyPics/2025-02-02-LCW-1.png)](/MyPics/2025-02-02-LCW-1.png){:target="_blank"}
Figure 1: starting point to create a Custom Extension
{:.figcaption}

We use a Managed Identity for authentication.


[![LCW_2](/MyPics/2025-02-02-LCW-2.png)](/MyPics/2025-02-02-LCW-2.png){:target="_blank"}
Figure 2: Set Parameter for the Custom Extension
{:.figcaption}

This is the final step in the Lifecycle Workflows dashboard, which directly creates
the Logic App in the Azure infrastructure.

[![LCW_3](/MyPics/2025-02-02-LCW-3.png)](/MyPics/2025-02-02-LCW-3.png){:target="_blank"}
Figure 3: All set and we can go ...
{:.figcaption}

[![LCW_4](/MyPics/2025-02-02-LCW-4.png)](/MyPics/2025-02-02-LCW-4.png){:target="_blank"}
Figure 4: Custom Extension is created... next steps are waiting in the Logic App Configuration
{:.figcaption}

Once the extension has been created along with the Logic App, the next step is to set up the Logic App. In our example, I created the Logic App directly from the Lifecycle Workflow dashboard, so the app is already pre-configured and we can get started right away. However, it is also possible to use an existing Logic App; in this case, some additional steps are required to deploy the app.
You can find more information on this in this Microsoft article.
https://learn.microsoft.com/en-us/entra/id-governance/configure-logic-app-lifecycle-workflows

The managed identity created with the Logic App must be initially authorized.

To do this, we retrieve the ID of the service principal, which you can find in the Logic App dashboard under “Identity.”

[![LCW_5](/MyPics/2025-02-02-LCW-5.png)](/MyPics/2025-02-02-LCW-5.png){:target="_blank"}
Figure 5: Logic App has it's own Service Principal
{:.figcaption}

For our example, it is sufficient to assign the Directory Reader role.

Depending on the actions performed by the Logic App, additional permissions may be required, as the workflow runs in the context of the service principal.

[![LCW_6](/MyPics/2025-02-02-LCW-6.png)](/MyPics/2025-02-02-LCW-6.png){:target="_blank"}
Figure 6: The Service Principal needs permissions according to the actions in the Logic App
{:.figcaption}

The created Logic App already has an HTTP action, in addition to the start trigger that waits for a manual invocation by the workflow.

In the next step, we add an action to define the “Audience” variable.
“Add an action…”

[![LCW_7](/MyPics/2025-02-02-LCW-7.png)](/MyPics/2025-02-02-LCW-7.png){:target="_blank"}
Figure 7: Designer view in a Logic App, an easy way to build a action sequence
{:.figcaption}


… and we continue by adding the corresponding action.

[![LCW_8](/MyPics/2025-02-02-LCW-8.png)](/MyPics/2025-02-02-LCW-8.png){:target="_blank"}
Figure 8: The ‘Variables’ action can be helpful
{:.figcaption}


Our variable is called “Audience”, and we assign it the URL to the Graph API.

[![LCW_9](/MyPics/2025-02-02-LCW-9.png)](/MyPics/2025-02-02-LCW-9.png){:target="_blank"}
Figure 9: ‘Variables’ action parameter options
{:.figcaption}


After saving, our app temporarily looks like this in the designer:

[![LCW_10](/MyPics/2025-02-02-LCW-10.png)](/MyPics/2025-02-02-LCW-10.png){:target="_blank"}
Figure 10: Our Logic App is growing ...
{:.figcaption}


In the next step, we adjust the HTTP action so that it retrieves all relevant user information
via the Graph API and give the action a meaningful name.

[![LCW_11](/MyPics/2025-02-02-LCW-11.png)](/MyPics/2025-02-02-LCW-11.png){:target="_blank"}
Figure 11: HTTP action! Powerful functionality to set HTTP requests
{:.figcaption}

Using “Insert dynamic content”, we append the URI with the username and, most importantly, select GET as the method.

With that, our HTTP action is complete and should look like this:

[![LCW_12](/MyPics/2025-02-02-LCW-12.png)](/MyPics/2025-02-02-LCW-12.png){:target="_blank"}
Figure 12: HTTP action will be used to get information from the specific user in our workflow via Graph API
{:.figcaption}

Now it gets exciting! We now add the “Data Operation” action, which processes
the result of the previously generated User Graph API call.


[![LCW_13](/MyPics/2025-02-02-LCW-13.png)](/MyPics/2025-02-02-LCW-13.png){:target="_blank"}
Figure 13: Data Operation action and "Parse JSON" can be used to grab information from the former API Graph Call
{:.figcaption}

In “Content,” we select the previous action, “HTTP Get User Information.”


[![LCW_14](/MyPics/2025-02-02-LCW-14.png)](/MyPics/2025-02-02-LCW-14.png){:target="_blank"}
Figure 14: Additional parameters for Parse JSON functionality
{:.figcaption}

At this point, a schema is required to type the provided output.
To do this, we use the option to generate a schema based on an example payload.

[![LCW_15](/MyPics/2025-02-02-LCW-15.png)](/MyPics/2025-02-02-LCW-15.png){:target="_blank"}
Figure 15: The Schema is an important essence in the parameter set
{:.figcaption}

The necessary Sample Payload to generate a schema can be obtained in the Graph Explorer, where we use the result of a GET request for a user account. We then copy the returned result to the clipboard.

Based on this Sample Payload, the schema will be created. Subsequently, the variants and attribute names can be used in further actions, as we will see later.


[![LCW_16](/MyPics/2025-02-02-LCW-16.png)](/MyPics/2025-02-02-LCW-16.png){:target="_blank"}
Figure 16: An example payload can be used to define the schema
{:.figcaption}


Then, we switch back to the Logic App and insert the result of the query here

[![LCW_17](/MyPics/2025-02-02-LCW-17.png)](/MyPics/2025-02-02-LCW-17.png){:target="_blank"}
Figure 17: Well done, schema imported
{:.figcaption}

After that, we have the schema generated from the payload and can save the action.

[![LCW_18](/MyPics/2025-02-02-LCW-18.png)](/MyPics/2025-02-02-LCW-18.png){:target="_blank"}
Figure 18: That's it - the Parse JSON action is complete
{:.figcaption}

In the next and final step, we create the action to send the email. An action suitable
for this is “Office 365 Outlook.”

[![LCW_19](/MyPics/2025-02-02-LCW-19.png)](/MyPics/2025-02-02-LCW-19.png){:target="_blank"}
Figure 19: The heart of our Custom Extension - send an e-mail
{:.figcaption}

This is where the magic happens! We populate the “Send Mail…” action with the
variables and content from the Parse JSON action, which in turn is derived from
the result of the Graph API call.

[![LCW_20](/MyPics/2025-02-02-LCW-20.png)](/MyPics/2025-02-02-LCW-20.png){:target="_blank"}
Figure 20: We use dynamic content to define the mail information for the new hire ...
{:.figcaption}

All the values that need to be filled can be inserted here using “Dynamic content.”

[![LCW_21](/MyPics/2025-02-02-LCW-21.png)](/MyPics/2025-02-02-LCW-21.png){:target="_blank"}
Figure 21: We get the mail-adress from the former Parse JSON
{:.figcaption}

This is how we can build our email, including the body, with the desired information.

Note that through the “Dynamic Content” browser, you can select all the contents
from the included actions. In our case, this is the Parse_JSON action, but you also have
the option to use content from, for example, the entry element “manual” in the email,
which was passed to the Logic App via the Lifecycle Workflow, depending on what you
need.

In the image below, for example, you can see the Manager—this wasn’t obtained from
the Graph API call, but was provided by the Lifecycle Workflow.

[![LCW_22](/MyPics/2025-02-02-LCW-22.png)](/MyPics/2025-02-02-LCW-22.png){:target="_blank"}
Figure 22: Our e-mail body is ready now
{:.figcaption}


Finally, we provide a callback back to the calling Lifecycle Workflow. Without this action,
a timeout occurs with the error message: “Callback not received within the timeout duration,”
and the workflow will have a failure status, even though the email was sent correctly.

Admittedly, here we statically return “OK”, which lacks any programming logic. Ideally,
you’d differentiate between success and failure, and return the respective status to the
Lifecycle Workflow. However, for the purpose of illustrating our example, this approach
should suffice.

[![LCW_23](/MyPics/2025-02-02-LCW-23.png)](/MyPics/2025-02-02-LCW-23.png){:target="_blank"}
Figure 23: Last but not least - we must send back the information to the Workflow, that we are done with the task
{:.figcaption}

That’s it for now. Our Custom Extension can now be used within your Lifecycle Workflows, just like any other tasks.

Additional information:
This setup is like a Lego set – it can be expanded and customized as needed.
Jan Bakker, in his example, sent an SMS and included a TAP. His post was great and inspired me to build my example. Thanks for that!

But these are just ideas of what you can do with this functionality, and it is meant to inspire you to implement your own requirements using Lifecycle Workflows.

Our example email, sent to the new employee’s private email address, looks like this:

<div style="border: 2px solid #000; max-width: 40%; height: auto; margin: 0 auto;">
  <a href="/MyPics/2025-02-02-LCW-24.png" target="_blank">
    <img src="/MyPics/2025-02-02-LCW-24.png" style="width: 100%; height: auto;" />
  </a>
</div>
Figure 24: This is how our e-mail looks like
{:.figcaption}

“I hope this has helped you get a better understanding of the aspects of Lifecycle Workflows, Custom Extensions, and Logic Apps. 

Enjoy trying it out 😀

... and ket me know, when you have hints, problems, questions, ...

Cover designed by vectorjuice / Freepik

{% include  share.html %}


