---
layout: post
title:  "Null in SQLite"
date:   2019-03-03 23:37:01 +0800
categories: [SQLite]
tags: [SQLite]
---
**null** is the absence of a value: **null** is not nothing, **null** is not something, **null** is not true, **null** is not false, **null** is not zero, **null** is not an empty string. Simply put, **null** is resolutely what it is: **null**.  

![AND_and_OR_with NULL](/assets/AND_and_OR_with NULL.png)

{% highlight sql %}
select true and null
select true or null
select false and null
select false or null
{% endhighlight %}
Test on mac os with DB Browser for SQLite, syntax error 'no such column: false: select null and false'  

{% highlight sql %}
select null and null
select null or null
{% endhighlight %}
Return null


Detecting the presense or absense of **null** is done using the **is null** or **is not null** operator. If you try to use any other operator, such as equals, greater than, and so on, you will receive a shock.  

Always remember that null is not equal to any other value ,even null itself. You cannot compare null with a value, and no null is greater than, smaller than, or in any way related to any another null.

sqlite> select null is null;  
1  

sqlite> select null = null;  
  

If you use **null**, you need to take special care in queries that refer to columns that may contain **null**
in their predicates and aggregates. **null** can do quite a number on aggregates if you are not careful.





[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
