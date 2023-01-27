---
title: So, What is Performance-Driven Development in PHP?
description: Performance is often left until you've shipped production. But why?
author: james-seconde
published: true
published_at: 2023-01-27T10:07:39.368Z
updated_at: 2023-01-27T10:07:39.403Z
category: tutorial
tags:
  - php
comments: true
spotlight: false
redirect: ""
canonical: ""
outdated: false
replacement_url: ""
---
During my years of Web Development experience, I have seen some astonishing code. While I don't exactly regard myself as some sort of code-wizard myself, I do have an understanding of some of the fundamentals that we need to know to write clean and efficient applications.

One of the biggest challenges we have faced in our PHP corner of the industry is getting developers to write using Test-Driven Development (TDD). The good news here is that things do certainly appear to be changing as far as I can see - especially within startup and agency organisations.

So, our work here is done, right? (obviously not!)

[In a talk I gave about Xdebug](), I alluded to the need for "Performance-Driven Development". It's time to revisit this to honour my promise of digging deeper into the subject.

If we write code in order to make it more robust by testing - why aren't we doing the same approach when it comes to performance? Who has heard of PDD? The behaviour in professional software development that I have seen always harks back to the cliche that developers consider

* Security
* Performance

...last. You've got deadlines. You need to ship it. It works. We'll worry about those two bullet points _later._ It's time to start looking at an example.

One of the aspects of development in PHP that I have seen the most is developers hammering at ORMs in [Symfony]() and [Laravel]() "until it does what I want", without thinking of the underlying SQL query and mixing in `forEach` loops at every possible opportunity. Relationships, when not architecturally considered, can easily create [n+1 problems]() when using queries. You'd think this wouldn't be too commonplace, but as I've stated from experience: this is everywhere.

For an example, consider the following Laravel code:

```php
public function getDashboardData()
{

	$dashboardData = [];
	
	foreach (Contact::all() as $contact) {
	
	   $settings = $contact->settings()->get();
	
	   foreach($settings as $setting) {
	       if ($setting->setting_name === 'CanPhone') {
	           if ($setting->setting_value === '1') {
	               $dashboardData['canPhone']['contact'][] = $setting->contact()->with('settings')->get()->toArray();
	           }
	       }
	   }
	}
	 
	return view('dashboard', ['data' => $dashboardData]);
```

Our _starting point_ before we can really drill down into the mechanics of how developers use the ORM is `Contact::all()`. I have seen things like this everywhere - in the developers' head, the easiest way to compile what data this dashboard puts out is to loop through every entity there is in a table, so get everything. In terms of table scans happening in MySql, this means you are already fetching an entire table _and then_ executing more queries _on top of that_ to get the `Setting` relation. In this case (and this is an anonymised real-life chunk of code, I promise you) the `Setting` relation _then makes a call to pull out the contact again_, despite it already existing within the context of the loop.

Examples such as these might not be ones an experienced developer would write - but that doesn't matter. The fact is, you might inherit or consult on a full monolith that someone else wrote. If you're not experienced, it's possible you came straight into application development without first understanding intricately how SQL _actually works_ (in more complex queries, I don't to be honest). So, n+1 problems will happen if you cannot code an ORM to use SQL concepts like `LEFT JOIN` and `INNER JOIN` effectively.

We have plenty of tools to help you discover problems like these - [Symfony has its excellent debug bar](), [Laravel has one as well](), Xdebug comes with a powerful profiler and Tideways now owns Xhprof. The problem I see is that developers from the bottom up tend to think of performance last when it comes to the everyday business of writing features, instead of using these tools _as part of the process of writing code and submitting it for Pull Requests._ In a way, this is quite similar to the cliche that Developers think of security last - when really, a more ideal solution would be to have all of your team consider both security *and* performance when writing PHP code.

I spoke with [Blackfire]() and [Platform.sh]() Developer Relations Engineer [Thomas di Luccio]() on what we should be considering when writing applications that consider performance as a "first-class" concern. The main argument was, he said, that we as developers need to consider performance from the absolute base level when writing code, and we don't do enough of it.

This was apparent when I worked on an application many years ago that took an all-too-familiar approach. Instead of considering SQL statements and how much work various classes that used SPL functions such as `foreach()` and `array_map()` were doing, coupled with no control over the underlying framework (which was proprietary, so with little control over the ORM), [Fastly]() was bolted onto the front end. Yes, the problem is solved. But the underlying codebase was still chock full of tech debt.

It's time for solutions. Taking our `getDashboardData()` example above, we need to fix it. Before we fix it, we need to profile what is happening when we access that dashboard page. Running the Laravel debug bar outputs the following:

**![](https://lh4.googleusercontent.com/Mlt5dIGmwZBhUmrcLRcsWSC4RSOz3wF3oi3ZSYx-wRzcQRZRKyUsN-tv4mQTD1S6G_gISbMvLlkHNvhLFmpUiAg-izbNYShgQF0BG4CGAsWrZGuyIHX-r9bXEnw9WJpi6TJHDPV8JJ1Jd9o9EykpQTR3JsN2DYLsRX-h2-Xf0pbi1edbFdD1Wuwb8q4p66h9)

Two hundred and eighty-six SQL queries for the dashboard to load - in 11.92 seconds. How does this happen? 

Well, it's a classic n+1.

On the developers' machine, the dashboard takes 1 second to load because there are a handful of fixtures in the database. As soon as this code hits production: this happens.

The thing is, if you have these tools enabled for your teams' developer environments, these sorts of occurrences can be made a thing of the past. The first step to controlling problems like this is *setting your thresholds.* Tooling like [Blackfire]() and [Tideways]() enables you to set thresholds and even integrate into your pipelines, but whichever approach you take (i.e. free tooling or enterprise products) you'll still need to have expectations of what is acceptable for your application when developers code it.

Let's say we want the target here to be less than 5 queries and a load time of under a second.

A more experienced developer checks the code, spots the n+1 problems, and educates the junior on how to get the data in one hit. Here's the result:

```php

public function getDashboardData()
{
	$dashboardData = Contact::with('settings')
	   ->whereRelation('settings', 'setting_name', '=', 'canPhone')
	   ->paginate(10)
	   ->toArray();

	return $dashboardData;
}
```

Much cleaner. But the important bit is: how does this look in the debug bar?

**![](https://lh3.googleusercontent.com/e4xDYMMIMqRVgHtswmTwhF113qMqn3jdsCb8ew8F-mJTUPTiLqShDoc5wPqCeORSf1kTWUXnGc-IN9sAJti7qmKbVwy1xDtc34iuV9q0ZhO4wATaQPmyOs6eE4qYczq2dd79lZ6ePTHXzN60GcN8tJBabA=s2048)

Four SQL statements, 324ms to load.

The lesson from this article isn't "here is obviously bad code, I fixed it, and the performance is better". *You* might probably see that the code was poor and that it needed fixing, sure. It is that if you are in a leadership position, the Symfony and Laravel debug bars should be enabled and used *as part of your development process* so that all of your team use it. You can set a rough threshold, introduce a test that forces a method to work much harder and assess the load time. We have the tooling to do this, so why should it be a secondary consideration?

What is Performance-Driven Development? It's the *process*. It's empowering your developers with the tooling they need _while they work on a feature._ Just like security, it should be a first-class citizen of the development cycle, and in the PHP world, we have absolutely top-class open-source tooling to deliver quality software.