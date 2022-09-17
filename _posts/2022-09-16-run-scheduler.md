---
layout: "post"
subtitle: ""
background: '/img/posts/world-athletics-webscrape/alessandro-venturi-XW_2N0Z9jmo-unsplash.jpg'
---

### Motivation
I have former teammate who lack a coach and wanted optimal training unrestricted by traditional 7-day cycles. I figured that scheduling training runs is a discrete optimization problem, so I used Google's Operations Research tools to model this.

#### Initial Challenges
There are few examples within the Google-or tools documentation. Many of the method parameters are vague, so I would often have to trace through the source code to figure out how to use it. This and some old conference videos were my main resources it figuring all of this out.

#### Constraints
Creating scheduling constraints is all based on one simple concept: the sum of boolean variables. Each boolean variable represents a distinct state in the schedule: day, cycle, session. Whether that variable is one or zero will be decided by the modeler when searching for solutions. 

```python
for c in all_cycles:
    for d in all_days:
        model.Add(sum(calendar[(c, d, s)] for s in all_sessions) == 1)
```

All of the constraints I used fell into some variation of this format. This constraint ensures that for every day on a calendar only one session is chosen. That is, only one session can have the value 1 while the rest must have the value 0. 

The more specific the constraint the more complex summation I had to write. For example, the following code satisfies some training preferences. You can't have a workout after a rest day or a rest day after a rest day, but you can have rest day after a workout.

```python
rList = [3] + workoutList
for c in all_cycles:
    for d in all_days:
        model.AddAtMostOne([calendar[(c, d, 3)]] + [calendar[next_day((c, d, r), 1)] for r in rList])
```

If you would like to see the full code, please check [github][1].

[1]: "https://github.com/treydur/run_scheduling" "Run Scheduler"

### Output
<pre>
cycle:0
	 day:0     rest
	 day:1     e 	 strides
	 day:2     e 	 strides
	 day:3     LR
	 day:4     e 	 strides
	 day:5     THRESH
	 day:6     e 	 hill
	 day:7     THRESH
	 day:8     e 	 hill
	 day:9     THRESH
</pre>

### Remarks
With more time I would like to continue to make this model more complex. It would be interesting to take into account periodization, mileage, and athlete preferences to intelligently create a feasible schedule.