#Filters#

Grease includes one filter that helps with calls from javascript. If you want all responses from Umbraco API calls to return camel-cased objects, you can set up the appropriate settings.

However if you only want *one* particular controller or even one *method* to do so, you can use the following filter:

```c#
using G42.UmbracoGrease.Filters;
using Umbraco.Web.Mvc;

namespace MyNamespace
{
    public class MyApplication : UmbracoAuthorizedController
    {
        [System.Web.Http.HttpGet]
        [CamelCasingFilter]
        public object MyMethod()
        {
            return new
            {
                MyProperty = "foo"
            };
        }
    }
}
```
>To make the camel-casing apply to the whole controller, add just above the `class` declaration

Output from this will be a camel-cased JSON string instead of Pascal-cased (i.e `myProperty` instead of `MyProperty`).