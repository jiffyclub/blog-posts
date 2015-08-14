*Note: I wrote this to describe agile/scrum development to my colleagues
because on our team I have the most experience with it,
but I'm not really expert on agile or scrum.*

Developing a product involves a number of different roles that all need to
coordinate with each other.
There are designers, sales people, engineers, writers, managers, and much more.
All those people need a way to track what they need to do
and what other people are doing.
Agile work-flow is a communications system that helps teams broadcast
what needs doing and what is being done.

### Get Started

Everything starts with defining the tasks at hand,
first at a high level and then progressively becoming more detailed
until you're describing manageable, understandable tasks.
For example, you might have a goal of implementing a comment system
where one task is "Add HTML escaping to user input".
Going from a high level goal to those manageable tasks can take a lot of
discussion before you get to the action,
but one of the valuable things about sticking to an agile work-flow is that
your team is forced to explicitly delineate the tasks ahead.

Tasks are the core unit of agile.
Tasks describe a specific item of work to be done and include an estimate
of the effort required to complete the task.
When dividing work among the team you assign tasks to the person
who will complete task.
When planning you will categorize tasks into those you want done soon
and those you want done later.
When you start working on a task you will indicate that it's "in progress"
so your teammates know what you're doing.
When you finish a task you'll mark it done so your teammates know it's
taken care of.

### Organizing

Once you start making tasks you'll quickly end up with lots of them,
requiring your team to think about which ones to work on right now.
In the popular [scrum][] work-flow teams work in [sprints][] of
two to four weeks, so tasks you want done soon are moved from the
global pool of tasks (the "backlog") to the next sprint.
You'll choose which tasks to target based on your development priorities
(are you trying to get that commenting system done?) and their severity.
An important part of this is that you only include as many tasks as
you expect to be able to finish
(so that you can effectively prioritize and not overload your team).
That's where the estimation comes in.
Tasks you don't finish in a sprint don't automatically move into the next sprint,
but if they are still a priority maybe they will.

### Task Estimation

To estimate a task means to assign an arbitrary number to it that reflects
the amount of effort/time required to complete the task.
It's not important that the numbers have any actual correlation with reality,
but is important that the numbers are consistently applied to similarly
sized tasks because you'll end up using those numbers to measure how many
units of work your team completes in a sprint.
Knowing how many units of work your team typically completes allows you
put the right amount of tasks into a sprint.
Putting estimates on tasks also allows you to check that you are assigning
the same amount of work to different team members.

It's okay for estimates to be wrong, and they should be very rough in order
to reflect how approximate estimates really are.
Hopefully the ones you overestimate will balance out the ones you underestimate.
I think of estimation as categorizing tasks into tiny, small, medium, and large.
But because these need to have numbers I'd suggest something like 1, 2, 4, and 8.
(If you've got a "huge" task it can probably be broken down into smaller tasks).

### Coding Tasks

As a programmer your development work-flow also fits into agile.
Assuming you're using something like [Git][] and [GitHub][] it works wonderfully
to have one branch and one pull-request per task.
That way all of our commits on a branch work toward completing one task
and all of the discussion on the task can reference one PR
(and vise-versa).
As soon as you complete a task open a PR for it, and in fact you might even
open a PR as you work so that people can more easily see your work-in-progress.
If you're lucky you'll be able to set up integration between your issue
tracker and your code repository (or maybe they're the same)
so that it's easy to point from task to PR and PR to task.
Very occasionally you might complete a couple tiny tasks in one PR,
but it won't happen often.

### Communicate

No development tool or work-flow will save you from talking to your teammates,
and in fact agile is *all about* talking to your teammates.
Make sure you have good communication tools and practices set up.
Ask each other questions,
send each other pictures,
do daily stand-ups,
draw things on white boards,
do code reviews,
have fun,
write everything down (maybe in a wiki),
use emoji,
and don't do anything in secret.
I love [Slack][] for team communication because it allows people to talk
to each other in a place their teammates can all see without being obtrusive
(unlike an email list).

Happy working!

[scrum]: https://en.wikipedia.org/wiki/Scrum_(software_development)
[sprints]: https://en.wikipedia.org/wiki/Scrum_(software_development)#Sprint
[Git]: http://git-scm.com/
[GitHub]: https://github.com/
[Slack]: http://slackhq.com/
