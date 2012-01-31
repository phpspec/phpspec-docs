Preface
=======

It was 2006 when I decided to learn the Ruby language (a fact that
probably isn't the best start to a PHP focused book but hey, it's where this
all started!). Around that time an alternative to using xUnit libraries for
Test-Driven Development (TDD) had emerged and was receiving a lot of
attention. Not one to turn down the tools available I installed the RSpec
framework and was introduced to the concept of Behaviour-Driven Development
(BDD), an evolution or repackaging of TDD whose focus was largely about
creating a new language to replace the traditional TDD terminology which was
focused on testing.

To my surprise, I found BDD fascinating. It made all the objectives of
TDD simpler to grasp and understand and my productivity using TDD properly
was a lot higher. It is not an easy concept to explain but BDD made TDD
easier and more enjoyable. For me, since I write code these days in my spare
time a lot, an enjoyable process was very much preferred.

Forward to 2007, and I found myself still preferring to use PHP. Ruby
is a very nice language but most of my interests were better served by a
language I had years of experience using. I was simply more productive using
PHP, I enjoyed it's sense of community, and I admit I had a Java-esque
affection for its syntax. Of course, I ran into one problem - PHP has no
RSpec-like framework. Say what you like about Ruby, but its community has an
annoying habit of introducing new practices and tools a lot faster than PHP
ever has.

So, after returning to PHP, I found no direct BDD support. I could
have simply molded my SimpleTest practices to a similar approach but it
would have barely been a half-measure. I wanted an RSpec like tool. I wanted
the ZenTest Autotest with it's utterly addictive Snarl or KDE Notify popups.
And I wanted it without catering to the expectations of xUnit frameworks.
And so, in October 2007, after a few discussions with community members like
Travis Swicegood, I became determined to just go and write one for PHP
rather than sit around bemoaning it's loss.

PHPSpec is the result of this effort. A Behaviour-Driven Development
(BDD) framework for PHP5.