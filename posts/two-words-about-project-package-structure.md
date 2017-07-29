#Project-Package Structure
WIP
##Why?
Jumped into #go-kit channel in slack, then jumped into go-kit github in order to read their examples. No notes on packages or project structure! This lead to some intresting discussions at slack.
##Foreword
I'm sure most of the newcomers will read some project structure guide or article within their first 2 weeks in the language. Everybody has read this [Medium post on standard package layout](https://medium.com/@benbjohnson/standard-package-layout-7cdbc8391fc1#.ds38va3pp); If you haven't, you should go there and read it immediately.
##No Examples
As a newcomer, I did not enounter any official guide regarding project structure. The only resources I found (besides googling) was those @gopher newbie resources whispered to me. Me and other people have questions that were not able to be resolved without human interaction. I'll try to sum up everything I've learned here.

> This issue seems similar to the problem Maven tried to fix, coming after Ant: "standarizing" the project and package layout. Is __go dep__ and similar tools going to be deeply integrated into IDEs and evolve into layout standarizers? Who knows.

Anyway, let's get started:

1. Package by dependency is a good concept, but do not enforce it on everything. I sometimes pull out packages by the names __util__ , __misc__ , __models__ etc. These are meant to be small though! Don't overdo it. One thing I also dislike is having package name conflicts (having __http__ subpackage, for example). Just change it a bit (__hypertext?__)to avoid conflicts if for some reason you need to drift away from having to cram all your dependent stuff in that package.

2. A single mock package is very good practise, even if you use external libraries to do mocking.

3. 2, with your main package bootstrapping every service you may need, makes testing a lot easier :) .

4. Keep your main packages under `cmd/{name}/main.go` in order to let other parts of your project be used as a library.

5. This one is for the monolith who would want to be chopped up into microservices in the future. Tightly-coupled dependencies should go under __internal/services/{name}__, while code that could be a separate library or be integrated into more projects, should live under __internal/platform/{name}__. You could stick to __internal/{name}__ for some other (edge-case) chunks of code but avoid it if you can (?), because it may evolve into chaos.

6. Never, ever have your unit tests in a separate directory from the thing you are testing. Also, don't move all the integration tests on a single test package/folder; place them at the correct scope (regarding to what they are testing).

Helpful extra link: https://stackoverflow.com/questions/14867452/what-is-a-sensible-way-to-layout-a-go-project
