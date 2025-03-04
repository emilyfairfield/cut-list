# cut-list
In 2019, my family and I built a murphy bed with bookcases in our home. We painstakingly put together our Bill of Materials (BOM) for the project, and found that many of the boards could be cut from large sheets of plywood. 

I searched for an online tool, which, given my BOM, would tell me how many sheets of plywood to buy, and subsequently, how to cut them, with the goal of minimizing cost. I found a tool that was close to what I needed - it told you how to cut stock boards to get your BOM, with the goal of using the fewest boards. However, it assumed you had these stock boards on hand and required that you input their quantities and dimensions.

I made due with that tool at the time just to get the project done, but I swore I would improve upon it. This is my attempt at doing just that.

First, let's see if we can formulate the problem as a Mixed Integer Linear Program (MILP):