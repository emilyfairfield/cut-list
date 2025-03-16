# cut-list
## Motivation:
A few years ago, my family and I built a murphy bed with bookcases in our home. We painstakingly put together our Bill of Materials (BOM) for the project, and found that many of the boards could be cut from large sheets of plywood. 

I searched for an online tool, which, given my BOM, would tell me how many sheets of plywood to buy, and subsequently, how to cut them, with the goal of minimizing cost. I found a tool that was close to what I needed - it told me how to cut stock boards to get my BOM, with the goal of using the fewest boards. However, it assumed I had these stock boards on hand and required that I input their quantities.

I made due with that tool at the time just to get the project done, but I swore I would improve upon it. This is my attempt at doing that.

## Assumptions:
First, let's list our assumptions:
* The user only intends to cut the stock boards along their two largest dimensions. (eg, they will never cut/plane a 8' x 4' x 0.75" board down to a 8' x 4' x 0.5" board.) The consequence of this assumption is that we only consider cutting BOM items of a given thickness from stock items of the same thickness.
* Grain direction doesn't matter. In other words, the user does not mind if some of the grain "points" vertically on one board and "points" horizontally on another. The consequence of this assumption is that we don't have to consider - or even know - grain direction when "rotating" BOM items to fit them together on a stock board.
* The user does not care about which wood species their pieces come from. They simply want to minimize cost.
* All pieces in the BOM, and all pieces of stock wood, are rectangular prisms.
* The solution is feasible.

## Problem Formulation:
Next, let's see if we can formulate the problem as a Mixed Integer Linear Program (MILP):

### Objective Function & Decision Variables, Attempt 1:
> **Objective is to minimize cost:**\
> $min_{\left(q_j\right)}\left( \sum_{j=1}^&infin; p_j q_j \right)$  
> where:  
> $p_j:$ price of stock item $j$    
> $q_j:$ our **decision variables**, representing quantity of stock item $j$ to buy    

### User Inputs:
#### Bill of Materials (BOM):
> $a_i:$ length/max dimension of BOM item $i$  
> $b_i:$ width/mid dimension of BOM item $i$  
> $c_i:$ height/min dimension of BOM item $i$  
> (where $a_i \geq b_i \geq c_i$)  

> [!NOTE]
> For model simplicity, $i$ is one **instance** of a board with given dimensions. If you need 2 of the same board, its dimensions must be given twice.

#### Stock Boards Available for Purchase:
> $l_j:$ length/max dimension of stock item $j$  
> $w_j:$ width/mid dimension of stock item $j$  
> $h_j:$ height/min dimension of stock item $j$  
> (where $l_j \geq w_j \geq h_j$)  
> $p_j:$ price of stock item $j$  


> [!NOTE]
> For model simplicity, $j$ is one **instance** of a stock board with given dimensions. Require the user to give the information only once, but the tool should automatically duplicate it several times*.

*Needs to be defined

### Objective Function & Decision Variables, Attempt 2:
Based on the above, we need to come up with a reasonable upper limit for the quantity of each stock board required to fulfill our BOM. Because we are assuming feasibility, we know that in the worst case, we can only cut one of our BOM boards from each stock board we buy. Of course, we don't know right off the bat which size of stock board would be paired with each BOM item in this worst case. So, a conservative upper limit would be one of *each* type of stock board *per* BOM item.

We now update our objective function such that the upper bound of our summation is $j=m$ where $m$ is the number of different types of boards times the number of BOM items, $n$. Our objective is still to minimize cost, but $q_j$ is no longer a decision variable, but a function of our new decision variable, $u_{ij}$. Keep in mind that because $j$ represents one **instance** of a stock board of given dimensions, $q_j$ can only evaluate to 0 or 1. 

> **Objective is to minimize cost:**\
> $min_{\left(u_{ij}\right)}\left( \sum_{j=1}^m p_j q_j \right)$  
> where   
> $`u_{ij} = \begin{cases} 1 & \text{if BOM item i is cut from stock board j} \\ 0 & \text{otherwise} \end{cases}`$    
> $`q_{j} = \begin{cases} 1 & \text{if we need to buy stock board j to satisfy our BOM} \\ 0 & \text{otherwise} \end{cases}`$  
> $p_j:$ price of stock item $j$  
> $n:$ total number of BOM items  
> $m:$ upper limit of stock items = number of different types of board $* n$  

### Constraints:
#### 1. All BOM items must be cut exactly once / from exactly one stock board:
> **constr. 1: ** $\sum_{j=1}^m u_{ij} = 1  \forall i$  

#### 2. The thickness (smallest dimension) of each BOM item must match that of the stock item from which it's cut: 
We want to constrain our problem such that:

$`u_{ij} = \begin{cases} 0 & \text{if } c_i \neq h_j \\ \in \{0,1\} & \text{otherwise} \end{cases}`$  

How can this be expressed as an inequality / constraint? Let's consider some examples:

| $c_i$  | $h_j$ | Desired $u_{ij}$ Upper Limit |
| ------------- | ------------- | ------------- |
| 0.75  | 0.75  | 1 (or more)  |
| 0.75  | 0.5  | 0  |
| 0.5  | 0.75  | 0  |

Can we use the ratio of $c_i$ to $h_j$ to get the desired $u_{ij}$?

YES!

We need BOTH of the following:

> $u_{ij} \leq \frac{c_i}{h_j} \forall i,j$  
AND  
> $u_{ij} \leq \frac{h_j}{c_i} \forall i,j$  

Because if $c_i \neq h_j$, then one of the above ratios will be less than one, and since $u_{ij} \in \{0,1\}$, this will force $u_{ij} = 0$.

#### 3. If any BOM items are planned to be cut from stock board j, we must buy stock board j:
We want to constraint our problem such that, for a given $j$, if any $u_{ij} \forall i$ is set to 1, $q_j$ gets set to 1. Based on the definition of $u_{ij}$, we know that $u_{ij}$ will be either 0 or 1 for all values of $i$ and $j$. We also know that there will be $n$ total $u_{ij}$ variables for each value of $j$. Using similar logic to constraint 2 above, we require that:

> $q_j \geq \frac{\sum_{i=1}^n u_{ij}}{n} \forall j$  

If any $u_{ij}$ are equal to 1 for a given stock board $j$, then the numerator will evaluate to some value greater than 0. And, since we defined $q_j$ to be either 0 or 1, this will force $q_j$ to 1. The maximum value of the numerator is $n$, so the right side of this inequality will never evaluate to more than 1.

#### 4. BOM items cannot exceed the boundaries of the stock board from which they're cut:
Using the intentionally complex cut pattern example below, let's consider how to write this constraint in terms of our decision variables.

![](./images/example_cuts.png)  

We're going to need additional decision variables, to answer not just "Is BOM item $i$ cut from stock board $j$?" but also, "What is the cut pattern for stock board $j$?". 

To this end, let's create additional decision variables for each BOM item's x and y coordinates with respect to the stock board. Arbitrarily choose the BOM item's upper left corner to be the point represented by $(x_i,y_i)$, such that:  

> $x_i:$ x coordinate of BOM item $i$'s upper left corner with respect to the upper left corner of the stock board from which it's cut  

> $y_i:$ y coordinate of BOM item $i$'s upper left corner with respect to the upper left corner of the stock board from which it's cut

Looking again at the example cut pattern above, there's one more element to this problem that we haven't yet considered. To enable the model to rotate boards, we will also need a decision variable to indicate whether a BOM item is "rotated" with respect to the stock sheet from which it's cut. 

> [!NOTE]
> A BOM item will be considered "rotated" if its longest dimension is **NOT** parallel to the stock board's longest dimension. 

Call this additional decision variable $r_i$, where:

> $`r_i = \begin{cases} 1 & \text{if BOM item } i \text{ is rotated wrt its stock board} \\ 0 & \text{otherwise} \end{cases}`$ 

Using these new decision variables and our existing decision variables $u_{ij}$, we need to devise our constraints that prevent BOM items from exceeding the boundaries of the stock board from which they're cut. Using the above diagram:  

![](./images/constr3a_table.png)  

##### 4a. BOM items' width cannot exceed width of stock board:

![](./images/constr3b.png)  

But wait. This is too restrictive. We don't want the total width of all 5 BOM items to be $\leq$ the stock width. We only want that to be true of each of the following groups of boards because they're next to each other:

![](./images/constr3_groups.png)  

We need to incorporate a "next to each other" term into the constraint above.

![](./images/constr3a_scoot.png) 

Let's enumerate all the ways board 1 can be "next to" board 2:
![](./images/constr3c.png)

Wait. Let's simplify this by enumerating the ways 2 boards are NOT "next to each other":
![](./images/constr3d.png)

Let's use $t$ to represent "NOT next to each other", where $t=t_1 + t_2$ and $t_1$ and $t_2$ represent scenarios 1 and 2, respectively, from the photo above.  

First, we know we should constrain $t_1$ to be 0 or 1:  
> $t_1 \in {0,1}$  

We also want:  
$`t_1 = \begin{cases} 1 & \text{if } y_2 \geq y_3 \\ 0 & \text{if } y_2 \lt y_3\end{cases}`$  

Which can be rewritten as:  
$`t_1 = \begin{cases} 1 & \text{if } \frac{y_2}{y_3} \geq 1 \\ 0 & \text{if } \frac{y_2}{y_3} \lt 1\end{cases}`$  

Try:  
$t_1 \leq \frac{y_2}{y_3}$  
Does this accomplish what we want?  
$`t_1 = \begin{cases} 0 & \text{if } y_2 \lt y_3 \text{ as desired}\\ 0,1 & \text{if } y_2 \geq y_3 \text{not restrictive enough, combine with another constraint?}\end{cases}`$  

BUT WAIT: $y_3$ is a non-negative decimal number, and could be equal to zero, which would result in the fraction above having a denominator of zero. Since $y_3$ can never be negative, we can remedy this by simply adding 1 to the numerator and denominator:  

> $t_1 \leq \frac{y_2 + 1}{y_3 + 1}$  

What additional constraint can we create to force $t_1 = 1$ when $\frac{y_2}{y_3} \geq 1$?  

Try:
$t_1 \geq y_2 - y_3$    


##### 4b. BOM items' height cannot exceed height of stock board:

#### 5. BOM items cannot overlap each other:

#### 6. All u_ij must be a non-negative integer:

#### 7. Integer constraints:
$u_{ij}, q_j, r_i \in\{0,1\} \forall i,j$

#### 8. Non-negativity constraints:
$x_i, y_i \geq 0 \forall i$

## Future Work:
* Create GUI for tool.
* Enable user to specify grain direction preferences.
* Enable user to specify multiple desired wood species for one project without having to run the model multiple times.
* Pull candidate stock items from common hardware stores' APIs instead of requiring user to input their information.