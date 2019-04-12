![](https://github.com/Microsoft/MCW-Template-Cloud-Workshop/raw/master/Media/ms-cloud-workshop.png "Cloud Workshops")

<div class="MCWHeader1">
Continuous delivery in Azure DevOps
</div>

<div class="MCWHeader2">
Whiteboard design session student guide
</div>

<div class="MCWHeader3">
April 2019
</div>

**Contents**

<!-- TOC -->

- [Continuous delivery in Azure DevOps whiteboard design session student guide](#continuous-delivery-in-azure-devops-whiteboard-design-session-student-guide)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Step 1: Review the customer case study](#step-1-review-the-customer-case-study)
    - [Customer situation](#customer-situation)
    - [Customer needs](#customer-needs)
    - [Customer objections](#customer-objections)
    - [Infographic for common scenarios](#infographic-for-common-scenarios)
  - [Step 2: Design a proof of concept solution](#step-2-design-a-proof-of-concept-solution)
  - [Step 3: Present the solution](#step-3-present-the-solution)
  - [Wrap-up](#wrap-up)
  - [Additional references](#additional-references)

<!-- /TOC -->

# Continuous delivery in Azure DevOps whiteboard design session student guide

## Abstract and learning objectives 

In this whiteboard design session, you will learn how to design a solution with a combination of Azure Resource Manager templates and Azure DevOps to enable continuous delivery with several Azure PaaS services.

At the end of this workshop, you will be better able to build templates to automate cloud infrastructure and reduce error-prone manual processes. In addition, you'll create an Azure Resource Manager (ARM) template to provision Azure resources, configure continuous delivery with Azure DevOps, configure Application Insights into an application, and create an Azure DevOps project and Git repository.

## Step 1: Review the customer case study 

**Outcome**

Analyze your customer's needs.

Timeframe: 15 minutes

Directions: With all participants in the session, the facilitator/SME presents an overview of the customer case study along with technical tips.

1.  Meet your table participants and trainer.

2.  Read all of the directions for steps 1-3 in the student guide.

3.  As a table team, review the following customer case study.

### Customer situation

Tailspin Toys, the world's most innovative construction toy company, was established to make and sell what has become one of the world's leading integrated construction systems for children. From the living room to the classroom, it encourages people of all ages to imagine, build, and play. It is committed to the development of children and aims to inspire and develop the builders of tomorrow through creative play and learning. Its products are sold worldwide and can be virtually explored on its public website.

Tailspin Toys is based in the United States and is headquartered in Portland, OR.

Tailspin Toys saw the positive potential of the cloud and moved its IT infrastructure, with no significant re-architecture, into Microsoft Azure around six months ago. Now that its business is running successfully in the cloud, it has started a series of process improvements to become a more agile company with a specific focus on delivering frequent feature updates and fixes to its public website. Alex Montgomery, VP of sales and the head of its online business team, says that, "Even though our products are better, our competitors are generating more online sales than we are. For every feature that we deliver to our website, they have delivered 2 or 3. Our development processes are too cumbersome and slow for us to build quality code at that pace."

When it moved its existing systems into Microsoft Azure, Tailspin Toys decided to use the Azure App Service to host its public website which is written as an ASP.NET Model View Controller MVC application. For the database (back-end) tier, they chose the Azure SQL Database service for full Platform-as-a-Service agility.

**Public website and data layer**

Todd Culp is the enterprise architect at Tailspin Toys, and he is responsible for the application and development team's environments in Azure. He has also been tasked with changing the development processes for his team so they can be more agile and adaptive in the marketplace.

The public website is currently deployed as an ASP.NET MVC 4 Web App in an App Service Plan in the S3 tier. The development team is hosting its source control for the system in Visual Studio Team Services using a Git repository with branches set up for multiple environments including "development," "test," and "master." As they get close to a release point, the developers have a manual internal code review and QA process in which they debug the application on each other's machines and run through some use cases to test new and existing features in the application. When the build is ready, they manually deploy updates to the website through Visual Studio or FTP. Todd has been tasked with automating the entire process of testing, building, and deploying to the cloud for both the QA team, so they can fully test new builds in a realistic cloud environment, and for the developers, so they cannot deploy any builds that fail the test suite. The builds for the different environments should not affect each other, and there must be an easy way to promote a tested build to production.

Todd requires the development team to run unit and integration tests for every build, but there is no formal procedure or accountability over this process within the team. Also, he always makes the team sign off on the build before it goes live, but there have been some bugs that appeared in production that should have been caught by a software-based test before deployment. He thinks the developers are not consistently running all required tests and needs an automated way to enforce this before builds are deployed into the cloud.

When customers encounter problems on the website, the Tailspin Toys' help desk team records the issues and submits them as tickets to the development team. The developers often complain about the quality of the application logs and must resort to a live "reproduction session" to witness application behavior and exceptions (especially for JavaScript bugs or timeouts between application layers). This sometimes causes the help desk tickets to remain unresolved for lengthy periods of time.

Todd wants to improve the turnaround time for fixing these bugs, and he needs better logs for the developers. He needs a solution to gather new types of logs including browser errors and application dependency errors such as timeouts. Ideally, he wants to make the application logs searchable as well as implement an automated warning system that emails alerts when application behavior is unusually slow or problematic.

**Existing solution architecture**

![This is a diagram of Tailspin Toys existing solution architecture. On the left is a Visual Studio icon and the words "development," "test," and "master," which represents the development team hosting its source control in Azure DevOps with branches set up for development, test, and master environments. Three arrows point from these environments to a server and monitor icon on the right labeled Manual internal code review & QA on local developer machines. An arrow points from this icon to another Visual Studio icon on the right labeled Visual Studio/FTP, which represents developers manually deploying updates through Visual Studio or FTP. Two arrows point from this icon to two icons on the right: on the top, Azure App Service with Web App (S3 Instance), and on the bottom, Azure SQL Database (S1 Instance).](images/studentguide/media/image2.png "Existing solution architecture diagram")

### Customer needs 

1.  Be able to automatically and continuously deploy new software builds to the Azure App Service web app.

2.  Ensure that continuously deployed builds to the cloud do not interfere with the production copy of the solution.

3.  Identify an automated way of deploying to different environments for "development," "test," and "production" so that changes or deployments to one environment do not affect the others.

4.  Configure the automated builds to first require that a full series of unit tests pass before a deployment is started.

5.  Provide a search feature and visual dashboard for the application logs so the developers can more quickly resolve help desk tickets.

6.  Enhance the logged data from the front-end website to give the developers a more complete picture of the application's performance and behavior.

    - Browser information such as browser page load time and user activity per page.

    - Application dependency metrics such as request times and request failures for communication with the database or other services.

7.  Implement proactive diagnostics to generate automatic alerts for unusual application behavior including aberrant request response time, dependency response time, and page load time.

### Customer objections 

1.  We do not want to be locked in to a specific source control repository. We are evaluating GitHub and Azure DevOps and need to be able to change between them without frustrating rework.

2.  We do not want the developers to be able to make changes to the Azure resources even though they will have access to make source code changes.

3.  If developers can deploy directly to the cloud, will that expose us to the same quality problems we had before when untested code was promoted to production?

4.  How much of an impact will these process changes have on our development cadence? Will learning this place a new burden on the developers?

5.  Our developers are already having a challenge learning how to use Git; will adding a continuous deployment system on top of that slow them down and confuse them even more?  

### Infographic for common scenarios

![Common Scenarios for building a continuous deployment pipeline with Azure DevOps. Images include the icon for Azure DevOps, a Git repo, and a pipeline](images/commonscenarios.png "Common Scenarios for building a continuous deployment pipeline with Azure DevOps")

## Step 2: Design a proof of concept solution

**Outcome**

Design a solution and prepare to present the solution to the target customer audience in a 15-minute chalk-talk format.

Timeframe: 60 minutes

**Business needs**

Directions:  With all participants at your table, answer the following questions and list the answers on a flip chart:

1.  Who should you present this solution to? Who is your target customer audience? Who are the decision makers?

2.  What customer business needs do you need to address with your solution?

**Design**

Directions: With all participants at your table, respond to the following questions on a flip chart:

*Continuous Integration and Deployment*

1.  What available system should you use to automate software builds and deployments of the application?

2.  Explain how you can continuously deploy new builds directly to the cloud without interfering with the production site.

3.  Document how to integrate unit tests into the continuous delivery process so that when a test fails to pass, the deployment process is flagged and stopped.

4.  Explain how you can test a new build simultaneously with an existing build, like an A/B test.

5.  Why shouldn't we have multiple long-lived branches in source control?

6.  Create a plan on how to switch the source control location from Azure DevOps to GitHub.

*Enhance system logging functionality*

1.  Implement a solution that will enable the logs to be searchable and visible in an online dashboard.

2.  Implement a solution to enhance the application logs to provide more useful performance and application behavior details, specifically around browser metrics and application dependencies. Discuss which visualization, or dashboard, options exist for the log results. Existing App Service logs already cover these topics:

    -  **Detailed Error Logging**---detailed error information for HTTP status codes that indicate a failure (status code 400 or greater).

    -  **Failed Request Tracing**---detailed information on failed requests, including a trace of the Internet Information Server IIS components used to process the request and the time taken in each component.

    -  **Web Server Logging**---information about HTTP transactions using the W3C extended log file format.

    -  **Application Diagnostics**---trace messages as defined in the source code.

    -  **Deployment Logs**

**Prepare**

Directions: With all participants at your table:

1.  Identify any customer needs that are not addressed with the proposed solution.

2.  Identify the benefits of your solution.

3.  Determine how you will respond to the customer's objections.

Prepare a 15-minute chalk-talk style presentation to the customer.

## Step 3: Present the solution

**Outcome**

Present a solution to the target customer audience in a 15-minute chalk-talk format.

Timeframe: 30 minutes

**Presentation**

Directions:

1.  Pair with another table.

2.  One table is the Microsoft team and the other table is the customer.

3.  The Microsoft team presents their proposed solution to the customer.

4.  The customer makes one of the objections from the list of objections.

5.  The Microsoft team responds to the objection.

6.  The customer team gives feedback to the Microsoft team.

7.  Tables switch roles and repeat Steps 2-6.

##  Wrap-up 

Timeframe:  15 minutes

Directions:  Tables reconvene with the larger group to hear the facilitator/SME share the preferred solution for the case study.

## Additional references

|    |            |
|----------|:-------------:|
| **Description** | **Links** |
| Enable diagnostics logging for web apps | <https://azure.microsoft.com/en-us/documentation/articles/web-sites-enable-diagnostic-log/> |
| Monitor web app performance  | <https://azure.microsoft.com/en-us/documentation/articles/insights-perf-analytics/> |
| Azure Pipelines | <https://azure.microsoft.com/en-us/services/devops/pipelines/> |
| Azure Pipelines examples | <https://docs.microsoft.com/en-us/azure/devops/pipelines/index?view=vsts#azure> |
| Switch deployment slots in Azure web apps | <https://blogs.msdn.microsoft.com/devops/2017/04/10/considerations-on-using-deployment-slots-in-your-devops-pipeline/> |
| App service continuous deployment | <https://docs.microsoft.com/en-us/azure/app-service/deploy-continuous-deployment> |
| App service staging environments | <https://azure.microsoft.com/en-us/documentation/articles/web-sites-staged-publishing/> |
| Application Insights | <https://azure.microsoft.com/en-us/documentation/services/application-insights/> |

