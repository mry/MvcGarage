# An ASP.NET Core 1.1 MVC Web Application running on OpenShift Container Platform v3.3 (or above).

**Important Note:** This project assumes the readers have basic working knowledge of *Red Hat OpenShift Container Platform v3.3* (or upstream project -> OpenShift Origin) & are familiar with key underlying technologies such as Docker & Kubernetes.  Readers are also presumed to have hands-on experience developing web applications using Microsoft .NET and associated technologies.  The *My Garage* application is written in C#.  For that matter, readers familiar with any object oriented programming language should be able to follow along the instructions easily and deploy the ASP.NET web application to OpenShift CP.
For quick reference, readers can refer to the following on-line resources as needed.

1.  [OpenShift Container Platform Documentation](https://docs.openshift.com/)
2.  [Kubernetes Documentation](http://kubernetes.io/docs/user-guide/pods/)
3.  [OpenShift CP .NET Core S2I builder Image] (https://docs.openshift.com/container-platform/3.3/using_images/s2i_images/dot_net_core.html)
4.  [ASP.NET Core] (https://docs.microsoft.com/en-us/aspnet/core/)
5.  [.NET Entity Framework Core] (https://docs.microsoft.com/en-us/ef/core/)
6.  [SQL Server on Linux] (https://docs.microsoft.com/en-us/sql/linux/)
7.  [Visual Studio Code Overview] (https://code.visualstudio.com/docs/introvideos/overview)
8.  [Get started with VS Code and .NET Core on MacOS] (https://channel9.msdn.com/Blogs/dotnet/Get-started-with-VS-Code-using-CSharp-and-NET-Core-on-MacOS)

## Description
The *My Garage* application is a simple MVC (Model-View-Controller) Web Application that allows users to keep track of maintenance or repair work done on their vehicles.
The application uses the following technologies -

1. .NET Core 1.1
2. ASP.NET Core 1.1
3. .NET EF Core 1.1
4. SQL Server vNext CTP 1.2

## Steps for deploying the *My Garage* application on OpenShift CP v3.3 (or later)

### A] Create a new project in OpenShift CP using the web console.

1. Login into the OpenShift CP web console/UI and create a new project.  Use 'test-asp-net-sql' for project name and 'MyGarage' for display name.  Feel free to provide any meaningful description.

Next, we will deploy the SQL Server container.

### B] Deploy Microsoft SQL Server database container

1. The SQL Server container requires super user privileges (runs as user **root**) to run.  To satisfy this requirement, we will modify the *anyuid* security context constraint (SCC).  We will need to add the **default** service account to the list of users in this SCC. Use the OpenShift CLI command (shown below) to add the service account **system:serviceaccount:test-asp-net-sql:default** to the list of users.  Then save the SCC configuration.

 ```
 $ oc edit scc/anyuid
 ```
 * Bear in mind, you will need cluster administrator privileges to be able to modify an SCC.
 
2. In the OCP web console/UI, click on 'Add to project'.  Then click on 'Deploy Image' tab.  Within this tab, click on 'Image Name' field, enter text **microsoft/mssql-server-linux** and then hit search.  Leave the 'Name' field as is.  Add the two environment variables as shown in the screen shot below.

  ![alt tag](https://raw.githubusercontent.com/ganrad/MvcGarage/master/images/SQLServer-01.png)  

3. Next, click the *'Create'* button at the bottom of the web page.  This action will initiate the SQL Server container *deployment* process.  Behind the scenes, OpenShift will fetch the SQL Server docker container image from the docker registry (Docker Hub),  push the image into the integrated docker registry within OpenShift, create an image stream to associate the image with your project(/namespace) and then finally instantiate a container from this image.  This process should take approx. 3 to 5 minutes to complete.  As soon as the SQL Server container is successfully deployed, you should be able to view the container in the *Overview* tab in the left navigational panel as shown in the screen shot below.

  ![alt tag](https://raw.githubusercontent.com/ganrad/MvcGarage/master/images/SQLServer-02.png)  

4. Once the SQL Server container is up and running, we will create two database entities - a) A database user/login so that the *My Garage* application can connect to the database and store/retrieve pertinent information and b) A **repairsdb** database which will store all information entered by users of the application.  In order to create these entities, we will first install the SQL Server CLI utility/tool **sqlcmd**.  You can install this utility on your local machine or on one of the OpenShift cluster nodes.  Follow the instructions [here](https://docs.microsoft.com/en-us/sql/linux/sql-server-linux-setup-tools) to install the **sqlcmd** utility/tool.
