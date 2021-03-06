# _For loops_ vs. _apply_ in R
In this post, we'll compare the computational efficiency of `for` loops vs. the `apply` function in R. 

## Background
`for` loops are a way of executing a command again and again, with the value of one or more variables changed each time. Let's say we have data on a bunch of people for food-related spending over one week. (And that yes, some days they spent only $0.11 and they never spent more than $10. Bear with me.) The first four columns of our matrix look like this:

![](https://1.bp.blogspot.com/-k8urW3PwrNo/WWfNHpLzWEI/AAAAAAAABZQ/QT98q-Nq9Z4Hq-yFRpUL-lgVppNEIXX4ACLcBGAs/s1600/spent.png)

If we want to know the average amount each person spent that week, we can take the mean of every column. If our matrix is called `our.matrix`, we could type out: 

```r
mean(our.matrix[, 1])
mean(our.matrix[, 2])
mean(our.matrix[, 3])
mean(our.matrix[, 4])
...
```

But a simpler way to do this, especially if you have a lot of columns, would be to iterate through the columns with a `for` loop, like this:

```r
for(i in 1:ncol(our.matrix)){
    mean(our.matrix[, i])
}
```
Here, the value of `i` is being replaced with the number 1, 2, 3, etc. up to the matrix's number of columns. 

Conceptually, this is pretty easy to understand. `For` loops are straightforward and for many applications, they're an excellent way to get stuff done. (We'll use nested loops in the next section, for example.) But one of the first things a beginning coder might hear about `for` loops is that as soon as you get the hang of them, it's a good idea to move onto more elegant functions that are simpler and faster. (This [DataCamp intro to loops post](https://www.datacamp.com/community/tutorials/tutorial-on-loops-in-r#gs.DrVUdXM), for example, mentions alternatives in the same sentence it introduces for loops!) One such "elegant" command is called `apply`. 

```r
apply(our.matrix, 2, mean)
```

The `apply` command above does the same thing as the two preceding chunks of code, but it's much shorter and considered better practice to use. We're making our final script length shorter and the code above is easier to read. But what about what's going on under the hood? Does your computer find the above command actually faster to compute? Let's find out.

## Methods
At its core, our methods will involve creating a matrix, performing a calculation on every column in the matrix with either a `for` or `apply`, and timing how long the process took. We'll then compare the computation times, which will be our measure of efficiency. Before we get started, however, we'll address three additional points:

### 1. Examine the role of matrix size
If there are differences in computational efficiency between `for` loops and `apply`, the differences will be more pronounced for larger matrices. In other words, if you want to know if you or your girlfriend is a better endurance runner, you'll get a more accurate answer if you run a marathon than if you race to that plate of Nachos on the table. (Save yourself an argument about fast-twitch versus slow-twitch muscle and just agree that watching Michael Phelps YouTube videos basically counts as exercise anyway.) So as we pit our competing methods against each other, we want to give them a task that will maximize the difference in their effectiveness.

But the nice thing about coding is that it's often really easy to get a more nuanced answer to our question with just a few more lines of code. So let's ask how the difference in effectiveness between `for` loops and `apply` _changes_ with the size of the matrix. Maybe there's effectively no difference until you're dealing with matrices the size of what Facebook knows about your personal life, or maybe there are differences in efficiency right from the start. To look at this, **we'll keep the number of columns constant at 1,000 but we'll vary the size of each column from 2 rows to 1,000.**

### 2. Vary how difficult the computation is
Maybe our results will depend on what computation, exactly, we're performing on our matrices. We'll use a simple computation (just finding the mean) and a more complicated one (finding the mean of the six smallest values, which requires sorting and subsetting too).  

### 3. Minimize the role of chance
At the core of statistics is that there are innumerable random forces swaying the results in any data collection we perform. Maybe that bird preening itself just had an itch and isn't actually exhibiting some complex self recognition. Maybe the person misread the survey question and doesn't actually think the capital of Montana is the sun. One way we address this randomness we can't control for is through replication. We give a survey to lots of people; we look at lots of birds. One sun-lover is probably a mistake, but if everyone in the survey thought the sun was the capital, then we need to sit down and reevaluate how Montana is being portrayed in the media. So in our code, we'll run our simulation 10 times to account for randomness within our computer's processing time.

_[The code to carry out this posts is in this repository. It's called for-loops_v_apply.R.]_

## Results
### 1. Simple calculation
When we just want to know the mean of each column in a matrix, it looks like `for` loops actually outcompete `apply` for matrices larger than 200,000 cells. (The difference is at most 0.03 seconds per calculation... but still!) 

![](https://i.imgur.com/sUOOLUZ.png)

In the figure above, the gray and red colors correspond to `for` loops and `apply`, respectively. The lines are the mean values for 100 runs, and the shaded regions are our 95% confidence intervals (mean +/- 1.96 * standard error).

The y-axis corresponds to the time it took to perform the calculation. Lower values means the calculation was faster. As you move to the right on the x-axis, you're seeing the computational time for increasingly larger matrices. We see the gray and red lines diverge because differences in the computational efficiency of `for` loops begin to outweigh `apply`. For relatively small matrices, though, it looks like there's not much difference between our two methods. 

### 2. More complex calculation
Now we ask R to find the mean of the six lowest values of each column of a matrix. This requires R to first order the column values from smallest to largest, then pay attention to only the smallest six values, then take the mean of those values. According to our figure, we find any differences between `for` loops and `apply` to be negligible. 

![](https://i.imgur.com/eyzGTLC.png)

Maybe for incredibly large matrices, `for` loops are slightly more efficient, as seen by the shaded regions not overlapping at the far right of the figure. However, we should still be careful in interpreting this figure; while the error envelopes don't overlap for some matrix sizes (e.g. 600k cells, 1 million), they do for other matrix sizes (e.g. 700k, 825k). It's pretty unlikely that `for` loops are going to be significantly better for some arbitrary matrix sizes and not others, so what we're probably seeing is randomness stemming from differences in the background processes running on my computer when a particular run was going. (To try to minimize these, I ran this analysis after restarting the computer and had no other open programs.) 

Fifty replicates of our simulation doesn't seem to be enough to reject the idea that there's any difference between `for` loops and the `apply` function here. Let's rerun the code with more replicates. To save time, we'll focus on small and large matrices and skip intermediate sizes.

![](https://i.imgur.com/Y6sh5g0.png)

With the additional clarity we get when we run our simulation with 100 replicates, we can see that there don't seem to be significant differences between the two methods for small matrices, as we saw earlier. For large matrices, however, we see a fairly consistent superiority of `for` loops when the matrices are greater than 800,000 cells, meaning our original intuition seems to hold. This indicates that `for` loops are more efficient at performing this complicated function than `apply`, but only when the matrices are huge.

## Discussion
So we see that `for` loops often have a slight edge over `apply` in terms of speed, especially for simple calculations or for huge matrices. Great, but this leaves us with a few questions.

### 1. Why are for loops more efficient?
The results above came as a surprise to me because I'd always heard about how inefficient `for` loops could be. It turns out I'd missed the bit that should follow that statement: `for` loops can be inefficient, but not if you allocate your memory well. It would be particularly inefficient, for example, to do something like `data <- c(data, new_data)`, which basically forces R to recall the result of every previous iteration, for every iteration. If you look at the code in this repository, you'll see that we instead created empty matrices whose cells were filled with values. 

So we didn't mess up, but that doesn't fully explain why `for` loops are faster. According to [this thread on Stack Exchange](https://stackoverflow.com/questions/5533246/why-is-apply-method-slower-than-a-for-loop-in-r), a `for` loop can outcompete `apply` here because `for` loops use the vectorized indexes of a matrix, while `apply` converts each row into a vector, performs the calculations, and then has to convert the results back into an output. 

_[For those of you who are really into the technical details, [lapply is an exception](https://stackoverflow.com/questions/2275896/is-rs-apply-family-more-than-syntactic-sugar) and is actually faster than for loops because more of its calculations are brought down into the language C and performed there.]_

### 2. If for loops are generally faster, why bother with apply?
The figures in this post have shown that `apply` is usually a bit slower than `for` loops. But does a difference of 0.01 seconds, or even 0.1 seconds, matter? If you're performing an evolutionary simulation on huge populations for thousands of generations, and you're doing this multiple times with different model parameters... then sure, maybe those 0.1 seconds add up. But the clarity in reading `apply` commands is far more important, I would argue. If you're sharing your code with a collaborator, or writing code you might read again in the future, you want to be as clear as possible. Concise code is better than a sprawling mess.

Thanks for reading. If you have any suggestions for a fun R project, shoot me an e-mail.

Best,
Matt
