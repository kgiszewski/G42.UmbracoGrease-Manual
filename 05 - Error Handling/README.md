#Error Handling#

Built-in to Grease is the ability to send emails whenever a 500 error occurs. To enable you will need to do edit the `Global.asax` file on your web root to this:
```
<%@ Application Inherits="G42.UmbracoGrease.UmbracoApplications.GreaseUmbracoApplication" Language="C#" %>
          
```

You can disable it temporarily by adding this key to the Grease AppSettings `G42.UmbracoGrease:ErrorHandlerDisabled` with value of `1`.

Set `G42.UmbracoGrease:ErrorEmailFrom` to an email address that the errors will come from.

Set `G42.UmbracoGrease:ErrorEmailToCsv` to a CSV list of addresses that the errors will go to.

Set `G42.UmbracoGrease:ErrorRedirectTo` to `~/server-error` or wherever you'd like the user to be directed to.

##v1##
In v1, email reporting was removed in favor of Slack reporting. All keys are now handled via a dashboard instead of the deprecated `AppSettings`.
