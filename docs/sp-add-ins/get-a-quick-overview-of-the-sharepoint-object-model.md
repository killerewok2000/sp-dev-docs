---
title: Get a quick overview of the SharePoint object model
description: Get introduced to the content hierarchy, and client-side runtime and batching.
ms.date: 09/26/2023
ms.localizationpriority: high
ms.service: sharepoint
---
# Get a quick overview of the SharePoint object model

[!INCLUDE [sp-add-in-deprecation](../../includes/snippets/sp-add-in-deprecation.md)]

This is the fourth in a series of articles about the basics of developing provider-hosted SharePoint Add-ins. You should first be familiar with [SharePoint Add-ins](sharepoint-add-ins.md) and the previous articles in this series, which you can find at [Get started creating provider-hosted SharePoint Add-ins](get-started-creating-provider-hosted-sharepoint-add-ins.md#next-steps).

> [!NOTE]
> If you have been working through this series about provider-hosted add-ins, you have a Visual Studio solution that you can use to continue with this topic. You can also download the repository at [SharePoint_Provider-hosted_Add-Ins_Tutorials](https://github.com/OfficeDev/SharePoint_Provider-hosted_Add-ins_Tutorials) and open the BeforeSharePointWriteOps.sln file.

In this article you'll take a brief break from coding to get a quick overview of the SharePoint Client-side Object Model (CSOM). This model is large and well-documented in MSDN with reference topics, "how-to's", and code samples. In this article, we can only provide the tip of the tip of the tip of the iceberg. But even a very short introduction will make much of the code you see in this series a lot less mysterious.

## Content hierarchy

The following table shows the hierarchy of content in SharePoint and the CSOM classes that represent them. Each of these entities has children of the type just under it.

|                                      **Entity**                                      |    **Class**     |                                                                                                                                                      **Remarks**                                                                                                                                                       |
| :----------------------------------------------------------------------------------- | :--------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SharePoint on-premises farm or SharePoint Online subscription (also called a tenant) |                  | There is only limited programmatic access to this level in CSOM. There is no Farm or Subscription or Tenant class, for example. (SharePoint's server-side object model, which cannot be used in add-ins, enables programmatic access to these entities.)                                                               |
| site collection                                                                      | **Site**         | A collection of websites that are grouped together for mainly administrative reasons and to house SharePoint components, such as branded master pages or custom security groups, that can be applied to all the child websites. All websites belong to some site collection.                                           |
| website                                                                              | **Web**          | A set of pages and SharePoint components. Can have subwebsites.                                                                                                                                                                                                                                                        |
| list                                                                                 | **List**         | Document libraries and other kinds of file libraries are also at this level. A document library is a special kind of list in which each row includes an attached document, and the other columns are data about the document, such as its author, when it was last edited, and who has it checked out.                 |
| list item                                                                            | **ListItem**     | A row in a list—that is, a list item—with particular values in the fields of the row. Also has a type. See next row.                                                                                                                                                                                                   |
| list item                                                                            | **Content Type** | The type of a list item. These are represented by the  **ContentType** class. Each is basically a set of columns and metadata. The simplest is the built-in **Item** content type. All other content types are derived from **Item**. SharePoint includes many built-in content types, such as Event and Announcement. |
| column                                                                               | **Field**        | A  **Field** object includes not only information about the underlying data type, but also information about how the data is formatted and rendered on forms, such as the forms for creating, displaying, and editing specific list items.                                                                             |

You can programmatically create custom lists, content types, column types, and list items.

In addition to content, the CSOM gives you access to users, groups, roles and permissions, taxonomy, search, and more.

## Client-side runtime and batching

CSOM uses a batching system. Chunks of managed code are converted into XML and sent to the server in a single HTTP request. For every command, a corresponding server object model call is made, and the server returns a response to the client in JavaScript Object Notation (JSON) format.

SharePoint code on a client begins by retrieving a client context object that represents the current request context, including the identity of the SharePoint website (and its parent site collection), and through this context you can obtain access to CSOM objects. The following is the basic structure that you will see again and again.

```csharp
using (var clientContext = spContext.CreateUserClientContextForSPHost())
{
    // CRUD operation or query code goes here.

    clientContext.ExecuteQuery();
}
```

Note the following about this code:

- The `spContext` object is of the type **SharePointContext** and is defined in the SharePointContext.cs (or .vb) file that is generated by the Office Developer Tools for Visual Studio. This file can be modified, but it is not often that you need to do so. For most SharePoint Add-in projects, this file and the TokenHelper.cs (or .vb) file, which is also generated by the tools, effectively function as extensions of CSOM itself.
- The `clientContext` object is the CSOM type **ClientContext**.
- The **ExecuteQuery** method bundles up the CRUD operation code in an XML message that it sends to the SharePoint server. There it is translated into equivalent server-side object model code and executed.

There was an example of this pattern in the previous article of this series, in the `GetLocalEmployeeName` method shown here.

```csharp
private string GetLocalEmployeeName()
{
    ListItem localEmployee;

    using (var clientContext = spContext.CreateUserClientContextForSPHost())
    {
        List localEmployeesList = clientContext.Web.Lists.GetByTitle("Local Employees");
        localEmployee = localEmployeesList.GetItemById(listItemID);
        clientContext.Load(localEmployee);
        clientContext.ExecuteQuery();
    }
    return localEmployee["Title"].ToString();
}
```

Note the following about this method:

- The first two lines in the **using** block appear to get references to the list and the list item object. But actually when these lines execute in the SharePoint client-side runtime, they are simply translated into an XML format. The **ExecuteQuery** method sends that XML to the server.
- The **Load** method adds something extra to the message: it tells the server to send the specified object down to the client. The **ExecuteQuery** method receives this object (as JSON) and uses it to initialize the client-side `localEmployee` variable. Subsequent client-side code then references that **ListItem** object and its members. As you can see, the next line references the value of the item's `"Title"` field. This line would have thrown an exception if the **Load** method had not been called because the object doesn't really exist on the client-side until it is loaded.

## Next steps

In the next article, we get back to coding and learn how to [add SharePoint write operations to the provider-hosted add-in](add-sharepoint-write-operations-to-the-provider-hosted-add-in.md).
