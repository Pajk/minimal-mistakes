---
title: Go get private repository
tags: golang
excerpt: Allow Go tools to fetch private repos
---

First [visit GitHub](https://github.com/settings/tokens), create a new personal access token and grant it access to private repositories. Then run following command to force git client to use this token when accessing GitHub repos through https.

{% highlight bash %}

git config --global url."https://YOUR-TOKEN:x-oauth-basic@github.com/".insteadOf "https://github.com/"

{% endhighlight %}
