#NodeHelper#

`NodeHelper` is a singleton class that keeps track of nodes that of special importance to your site. This is especially helpful on Umbraco installs that have multiple sites with similar structure (white label).

To best illustrate what NodeHelper does, let's look at a case study.  Imaging you have a site in the following structure:

- Site 1
-- Home
--- About Us
-- Site Settings
--- Special Folder

- Site 2
-- Home
--- About Us
-- Site Settings
--- Special Folder

Normally if we needed a value stored on the `Site Settings` page placed on another template, we would have to use some sort of hierarchy method to traverse the tree. This requires some code like the following from the perspective of the `About Us` page template:

```
var someSetting = Model.Content.AncestorOrSelf(1).Descendants().First(x => x.DocumentTypeAlias == "SiteSettings").GetPropertyValue<int>("someSetting");  
```

This syntax works very well, but there are two (potential) issues:

1. The syntax is somewhat verbose
2. A large site with lots of content begins to take time when doing the lookups like this (even though it's searching the cached content)

You *could* write a helper method to get around the first issue, but the second issue is a little harder to overcome over time.

A different approach would be to cache certain nodes as *points of reference* that would make the template code more meaningful and using known values for the points of reference, the site runs faster*.

>*Sadly, no official benchmarking has been done to verify this claim.

So enter 'NodeHelper', the syntax to do the very same lookup would be a little cleaner:

```
var someSetting = NodeHelper.Instance.CurrentSite.SiteSettings.GetPropertyValue<int>("someSetting");
```

Aside from the boilerplate `NodeHelper.Instance`, the rest of the code illustrates what it is doing a little better (arguably of course). It's also a bit faster since under the hood the node Id of the `SiteSettings` node is stored in memory and no longer needs to be looked up on subsequent requests.

Additionally, instead of the magic call to `AncestorOrSelf(1)` (which means current site), we call `CurrentSite` which performs a more meaningful action.

##Configuration##
Out of the box, `NodeHelper` includes the following properties for each site included:

* Home
* Site Settings

This due to the default model named `Site` which can be inspected here: https://github.com/kgiszewski/G42.UmbracoGrease/blob/master/src/G42.UmbracoGrease/G42NodeHelper/Models/Site.cs

You can however add additional properties by extending the `site` class and overriding the `MapSite` method.  The following code adds a new member called `Redirects`:

```c#
using System.Linq;
using G42.UmbracoGrease.G42NodeHelper;
using G42.UmbracoGrease.G42NodeHelper.Models;
using umbraco.cms.businesslogic.web;
using Umbraco.Core.Logging;
using Umbraco.Core.Models;
using Umbraco.Web;

namespace MyNamespace
{
    public class MySite: Site
    {
        public int RedirectsId { get; set; }
        public IPublishedContent Redirects
        {
            get
            {
                return GetUmbracoContent(RedirectsId);
            }
        }

        public override Site MapSite(Site site, Domain domain, IPublishedContent rootNode, IPublishedContent siteRootNode, IPublishedContent siteSettings)
        {
            if (siteSettings != null)
            {
                LogHelper.Info<NodeHelper>("Site Settings Found, looking for redirects folder...");

                IPublishedContent redirects = null;

                redirects = siteSettings.Children().FirstOrDefault(x => x.DocumentTypeAlias == "RedirectsFolder");

                RedirectsId = (redirects != null) ? redirects.Id : siteRootNode.Id;

                LogHelper.Info<NodeHelper>("Found RedirectsFolder? " + (Redirects != null));
            }

            return site;
        }
    }
}
```

With the above in place, we can then use `NodeHelper` like so in a template:

```
IPublishedContent redirectsNode = ((MendozaSite)NodeHelper.Instance.CurrentSite).Redirects;
```

##But Singletons are Bad!##
Well that's what gets said at least due to the fact that they are harder to use with unit testing. Our experience is they are really nice persistently cached objects.

##Additional Notes##
Grease includes a custom section and one of the tree items is a way to view what domains are pointing to which nodes. You can also clear NodeHelper in real-time, it's thread-safe and is lazy by the fact it doesn't get created until it is first needed.

##Summary##
So to recap, `NodeHelper` is just syntactic sugar with a little performance boost. In the near future, you will be able to drop `NodeHelper.Instance` and possibly just do `CurrentSite.Home.GetPropertyValue<int>("someSetting");`