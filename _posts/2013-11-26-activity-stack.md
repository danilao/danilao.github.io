---
layout: post
title: Android tip. Remove an Activity from stack after going to another
tags: android, tip, activity, stack
comments: true
---

> This is a migrated post from my former blog.

This entry explains a way of *removing* and activity when you navigate to another. So, if you press back button you won't return to it.

But, is it useful?

![Activity stack scheme]({{site.url}}/img/activity-stack.png)

<!--break-->

&nbsp;

Imagine you're in a `LoginActivity` and after success, the apps brings you to the `MainActivity`. Then, you press the back button and return to the `LoginActivity`. To avoid it, add the next property to your `AndroidManifest.xml`:

{% highlight xml %}
<activity
    android:name="com.example.LoginActivity"
    android:label="@string/app_name"
    android:noHistory="true">
{% endhighlight %}
