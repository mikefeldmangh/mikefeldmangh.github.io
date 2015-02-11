---
layout: post
title: A (Not So Good?) Solution For A Complex Or In A Rails Scope
---

In my Ruby on Rails side project, I've been using the [Filterrific gem](https://github.com/jhund/filterrific) for filtering. And it's been terrific. But then I had to add a new filter and I just couldn't figure out how to get it to work. I searched online for a few days and found little things that helped me in the end but nothing that fit just right so I'm writing about what I came up with in the event it might help others.

## Models

I have three models with the following relationships:

{% highlight ruby %}
class Issue < ActiveRecord::Base
  has_many :votes
  ...
end

class User < ActiveRecord::Base
  has_many :votes
  ...
end

class Vote < ActiveRecord::Base
  belongs_to :issue
  belongs_to :user
end
{% endhighlight %}

## Process 

I didn't have much trouble writing the scope to filter by given votes for the current user. So you could find issues for which you voted with a yes or a no or both. But you couldn't find issues you haven't voted on yet. Here's the original scope:

{% gist mikefeldmangh/7a961f786b895a7927ae original_vote_types_only.rb %}

I was having trouble figuring out how to get the issues which didn't have votes for the current user. I could do it with SQL:

{% gist mikefeldmangh/7a961f786b895a7927ae no_vote_sql.rb %}

But I really didn't want to use SQL. And I had to combine with the existing where statement with an "OR". I looked back at the Filterrific documentation again and saw Jo Hund had made a major upgrade to version 2.0 since I started using the gem. He also added more [scope examples](http://filterrific.clearcove.ca/pages/active_record_scope_patterns.html) to the documentation. Great! I upgraded my gem and updated my code first. Some of his examples use Arel which I didn't know much about. I'm still kind of new to Rails. But it looked interesting. 

## Final Result

I'll cut to the chase and show what I finally came up with after a LOT of trial and error. But I kind of doubt this code is very efficient. And I also have several questions about it outlined below. 

{% gist mikefeldmangh/7a961f786b895a7927ae working_vote_type_scope.rb %}

## Questions 

#### Question 1 

I was able to come up with the `no_vote_relation` on line 35. But then I had to redo my where statement in my original vote types scope above to be an arel relation. This took a little longer and the `vote_types_relation` on line 29 works but it seems much more complicated. And I didn't think I should need to use an `in` for it. But it was the only thing I could come up with to get the join to work. Anyone have any suggestions for this? Or is it fine?

#### Question 2

Both the `no_vote_relation` and the `vote_types_relation` have a large portion of code that is identical between the two. So I tried to DRY out the code. See commented line 24. But when I tried to use that `issue_vote_join_relation` inside the other two (see commented lines 28 and 34), the scope method would get called about four times. That seems extremely bizarre to me. There's probably a reasonable explanation. Let me know if you know what it is.

#### Question 3

The simple `where` call at the end of the method works. But based on one of the Filterrific examples, I thought I needed to do something like this:

{% highlight ruby %}
where(
  Issue \
    .where(final_rel)
)
{% endhighlight %}

But that didn't work. Just using the `where` by itself was a complete guess. It just happened to work. I still don't understand going between arel relation and ActiveRecord::Relation. I need to do more research into this. If you have references or explanations, let me know.

## Conclusion

I hope this helps someone out there working on complex `or` queries with a join in a scope method since I couldn't find anything similar online. If you can answer any of my questions above or have anything to say about the efficiency of the code, please comment below. Thanks!
