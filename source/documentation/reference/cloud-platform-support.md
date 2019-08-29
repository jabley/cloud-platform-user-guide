## Cloud Platform Support

#### Background

The Cloud Platform Team has built a new platform to replace Template Deploy. We are employing modern tools and techniques to enable teams to develop and support their modern cloud native software and infrastructure.

This also provides us with an opportunity to enable teams, through self-service, to develop and support products the way they want to, without having to rely on the Cloud Platform Team for assistance.

This new flexibility changes the way in which the Cloud Platform team provides support to our users.

#### Cloud Platform Team Support Processes

The Cloud Platform [Operational Processes](#operational-processes.md) details how we will offer support to our users for the new platform.

This includes support hours, out-of-hours support, operational communication with users, incident management, and how we measure performance. 

#### What does Cloud Platform Team support

Below is a diagram of the Cloud Platform.

![Cloud Platform Diagram](../images/cloud-platform-diagram.png)

The Cloud Platform Team supports everything outside of a user’s namespace. Namespaces are shaded blue in the picture above.

Support includes:

* responding to failures of the underlying platform e.g. the cluster
* responding to failure of the management layer of the platform
* responding to failure of reusable components created by the Cloud Platform team and provided to users 
* responding to failures of the Kubernetes API 
* security of the platform  

#### What the Cloud Platform Team doesn’t support

The Cloud Platform Team doesn’t provide:

* any application related support, including responding to application-specific alerts or failures
* any support for infrastructure and other tools deployed within a users namespace
* security of the application
* ongoing maintenance of the application, including patching, updating dependencies, and ensuring it remains deployable
* user support for users of the application

#### What this means for Cloud Platform users

It means teams using the platform will be responsible for supporting the infrastructure they build, and the applications running on it.

We expect teams or service areas using the platform to ensure they can, for the life of the service:

* monitor and manage the health of their applications and service-specific infrastructure (setting up and responding to monitoring, logging, and alerts, as appropriate)
* keep their applications and service-specific infrastructure appropriately patched, up-to-date, and secure
* ensure their applications remain deployable end-to-end from source code
* regularly assess whether the service is still meeting user needs, and make appropriate efforts to improve, replace, or end-of-life the service if not

The Cloud Platform Team will be responsible for support of the underlying platform and “building blocks” that we make available to users.

The new Cloud Platform introduces modern technology that users may not already be familiar with. The Cloud Platform User Guide is intended to help users understand some of the basic concepts of working with Kubernetes, as well as providing links to useful training and reference materials. In addition, the Cloud Platform Team will be very happy to assist with any queries you might have at #cloud-platform-users slack channel.

You can also speak to your line manager or Head of Profession should you have any further training needs.

#### How does this impact products still using Template Deploy?

The Cloud Platform Team have been reaching out to teams since October 2018 to help plan migration away from Template Deploy.

In the interim, support for products using Template Deploy remains unchanged. Support will continue to be offered to these services under the existing Incident Response and Tuning (IRAT) process. 

To raise Template Deploy related support requests you should create a [Github Issue](http://goo.gl/msfGiS) with details of the problem.

As Template Deploy is being phased out, the Cloud Platform Team will not develop this product further and will aim instead to only fix incidents as they arise.

#### Where to find help

If you have any questions or require help from the Cloud Platform Team in relation to the new Cloud Platform contact us at the `#ask-cloud-platform` slack channel.

