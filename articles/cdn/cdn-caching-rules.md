---
title: Control Azure CDN caching behavior with caching rules | Microsoft Docs
description: You can use CDN caching rules to set or modify default cache expiration behavior both globally and with conditions, such as a URL path and file extensions.
services: cdn
author: duongau
manager: kumud
ms.service: azure-cdn
ms.topic: how-to
ms.date: 02/21/2023
ms.author: duau
---

# Control Azure CDN caching behavior with caching rules

This article describes how you can use content delivery network (CDN) caching rules to set or modify default cache expiration behavior. These caching rules can either be global or with custom conditions, such as a URL path and file extension.

> [!NOTE] 
> Caching rules are available only for **Azure CDN Standard from Verizon** and **Azure CDN Standard from Akamai** profiles. For **Azure CDN from Microsoft** profiles, you must use the [Standard rules engine](cdn-standard-rules-engine-reference.md) For **Azure CDN Premium from Verizon** profiles, you must use the [Verizon Premium rules engine](./cdn-verizon-premium-rules-engine.md) in the **Manage** portal for similar functionality.
 
Azure Content Delivery Network (CDN) offers two ways to control how your files get cached:

**Caching rules:** Azure CDN provides two types of caching rules: global and custom.

- Global caching rules - You can set one global caching rule for each endpoint in your profile, which affects all requests to the endpoint. The global caching rule overrides any HTTP cache-directive headers, if set.

- Custom caching rules - You can set one or more custom caching rules for each endpoint in your profile. Custom caching rules match specific paths and file extensions, get processed in order, and override the global caching rule, if set. 

**Query string caching:** You can adjust how the Azure CDN treats caching for requests with query strings. For information, see [Control Azure CDN caching behavior with query strings](cdn-query-string.md). If the file isn't cacheable, the query string caching setting has no effect, based on caching rules and CDN default behaviors.

For information about default caching behavior and caching directive headers, see [How caching works](cdn-how-caching-works.md). 

## Accessing Azure CDN caching rules

1. Open the Azure portal, select a CDN profile, then select an endpoint.

2. In the left pane under Settings, select **Caching rules**.

   ![CDN Caching rules button](./media/cdn-caching-rules/cdn-caching-rules-btn.png)

   The **Caching rules** page appears.

   ![CDN Caching rules page](./media/cdn-caching-rules/cdn-caching-rules-page.png)


## Caching behavior settings
For global and custom caching rules, you can specify the following **Caching behavior** settings:

- **Bypass cache**: Don't cache and ignore origin-provided cache-directive headers.

- **Override**: Ignore origin-provided cache duration; use the provided cache duration instead. This setting doesn't override cache-control: no-cache.

> [!NOTE] 
> For **Azure CDN from Microsoft** profiles, cache expiration override is only applicable to status codes 200 and 206. 

- **Set if missing**: Honor origin-provided cache-directive headers, if they exist; otherwise, use the provided cache duration.

![Global caching rules](./media/cdn-caching-rules/cdn-global-caching-rules.png)

![Custom caching rules](./media/cdn-caching-rules/cdn-custom-caching-rules.png)



## Cache expiration duration
For global and custom caching rules, you can specify the cache expiration duration in days, hours, minutes, and seconds:

- For the **Override** and **Set if missing** **Caching behavior** settings, valid cache durations range between 0 seconds and 366 days. For a value of 0 seconds, the CDN caches the content, but must revalidate each request with the origin server.

- For the **Bypass cache** setting, the cache duration gets automatically set to 0 seconds, which isn't a modifiable value.

## Custom caching rules match conditions

For custom cache rules, two match conditions are available:
 
- **Path**: This condition matches the path of the URL, excluding the domain name, and supports the wildcard symbol (\*). For example, _/myfile.html_, _/my/folder/*_, and _/my/images/*.jpg_. The maximum length is 260 characters.

- **Extension**: This condition matches the file extension of the requested file. You can provide a list of comma-separated file extensions to match. For example, _.jpg_, _.mp3_, or _.png_. The maximum number of extensions is 50 and the maximum number of characters per extension is 16. 

## Global and custom rule processing order
Global and custom caching rules get processed in the following order:

- Global caching rules take precedence over the default CDN caching behavior (HTTP cache-directive header settings). 

- Custom caching rules take precedence over global caching rules, where they apply. Custom caching rules get processed in order from top to bottom. That is, if a request matches both conditions, rules at the bottom of the list take precedence over rules at the top of the list. Therefore, you should place more specific rules lower in the list.

**Example**:
- Global caching rule: 
   - Caching behavior: **Override**
   - Cache expiration duration: One day

- Custom caching rule #1:
   - Match condition: **Path**
   - Match value: _/home/*_
   - Caching behavior: **Override**
   - Cache expiration duration: Two days

- Custom caching rule #2:
   - Match condition: **Extension**
   - Match value: _.html_
   - Caching behavior: **Set if missing**
   - Cache expiration duration: Three days

When you set these rules, a request for _&lt;endpoint hostname&gt;_.azureedge.net/home/index.html triggers custom caching rule #2, which get set to: **Set if missing** and 3 days. Therefore, if the *index.html* file has `Cache-Control` or `Expires` HTTP headers, they get honored; otherwise, if you don't set these headers, the file gets cached for three days.

> [!NOTE] 
> Files that are cached before a rule change maintain their origin cache duration setting. To reset their cache durations, you must [purge the file](cdn-purge-endpoint.md). 
>
> Azure CDN configuration changes can take some time to propagate through the network: 
> - For **Azure CDN Standard from Akamai** profiles, propagation usually completes within one minute. 
> - For **Azure CDN Standard from Verizon** profiles, propagation usually completes in 10 minutes.  
>

## See also

- [How caching works](cdn-how-caching-works.md)
- [Tutorial: Set Azure CDN caching rules](cdn-caching-rules-tutorial.md)
