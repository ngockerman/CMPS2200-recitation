# CMPS 2200  Recitation 03

**Name (Team Member 1):** Natalie Gockerman  
**Name (Team Member 2):**_________________________



## Analyzing a recursive, parallel algorithm


You were recently hired by Netflix to work on their movie recommendation
algorithm. A key part of the algorithm works by comparing two users'
movie ratings to determine how similar the users are. For example, to
find users similar to Mary, we start by having Mary rank all her movies.
Then, for another user Joe, we look at Joe's rankings and count how
often his pairwise rankings disagree with Mary's:

|      | Beetlejuice | Batman | Jackie Brown | Mr. Mom | Multiplicity |
| ---- | ----------- | ------ | ------------ | ------- | ------------ |
| Mary | 1           | 2      | 3            | 4       | 5            |
| Joe  | 1           | **3**  | **4**        | **2**   | 5            |

Here, Joe (somehow) liked *Mr. Mom* more than *Batman* and *Jackie
Brown*, so the number of disagreements is 2:
(3 <->  2, 4 <-> 2). More formally, a
disagreement occurs for indices (i,j) when (j > i) and
(value[j] < value[i]).

When you arrived at Netflix, you were shocked (shocked!) to see that
they were using this O(n^2) algorithm to solve the problem:



``` python
def num_disagreements_slow(ranks):
    """
    Params:
      ranks...list of ints for a user's move rankings (e.g., Joe in the example above)
    Returns:
      number of pairwise disagreements
    """
    count = 0
    for i, vi in enumerate(ranks):
        for j, vj in enumerate(ranks[i:]):
            if vj < vi:
                count += 1
    return count
```

``` python 
>>> num_disagreements_slow([1,3,4,2,5])
2
```

Armed with your CMPS 2200 knowledge, you quickly threw together this
recursive algorithm that you claim is both more efficient and easier to
run on the giant parallel processing cluster Netflix has.

``` python
def num_disagreements_fast(ranks):
    # base cases
    if len(ranks) <= 1:
        return (0, ranks)
    elif len(ranks) == 2:
        if ranks[1] < ranks[0]:
            return (1, [ranks[1], ranks[0]])  # found a disagreement
        else:
            return (0, ranks)
    # recursion
    else:
        left_disagreements, left_ranks = num_disagreements_fast(ranks[:len(ranks)//2])
        right_disagreements, right_ranks = num_disagreements_fast(ranks[len(ranks)//2:])
        
        combined_disagreements, combined_ranks = combine(left_ranks, right_ranks)

        return (left_disagreements + right_disagreements + combined_disagreements,
                combined_ranks)

def combine(left_ranks, right_ranks):
    i = j = 0
    result = []
    n_disagreements = 0
    while i < len(left_ranks) and j < len(right_ranks):
        if right_ranks[j] < left_ranks[i]: 
            n_disagreements += len(left_ranks[i:])   # found some disagreements
            result.append(right_ranks[j])
            j += 1
        else:
            result.append(left_ranks[i])
            i += 1
    
    result.extend(left_ranks[i:])
    result.extend(right_ranks[j:])
    print('combine: input=(%s, %s) returns=(%s, %s)' % 
          (left_ranks, right_ranks, n_disagreements, result))
    return n_disagreements, result

```

```python
>>> num_disagreements_fast([1,3,4,2,5])
combine: input=([4], [2, 5]) returns=(1, [2, 4, 5])
combine: input=([1, 3], [2, 4, 5]) returns=(1, [1, 2, 3, 4, 5])
(2, [1, 2, 3, 4, 5])
```

As so often happens, your boss demands theoretical proof that this will
be faster than their existing algorithm. To do so, complete the
following:

a) Describe, in your own words, what the `combine` method is doing and
what it returns.

.  The combine method begins with 2 already sorted lists, left_ranks and right_ranks.
.  Its goal is to combine the 2 lists, counting each time there is a disagreement (the right number is less than the remaining left numbers).
.  To do so, the function first defines 2 pointers as starting points in the lists, an empty list named result to hold the fully sorted list, and a variable to hold the count of disagreements found.
.  It then runs a while loop that will continue to run as long as both lists have numbers left to compare.
.  While running, each time the function finds a 
.  
.  
.  
.  

b) Write the work recurrence formula for `num_disagreements_fast`. Please explain how do you have this.

.  The work recurrence formula for this function is W(n) = 2W(n/2) + Θ(n).
.  This is found because the algorithm calls recursively in 2 halves, giving us 2W(n/2), 2 times the work of half the items.
.  We also know there is only one linear-time merge because when combining them together, every card is looked at once, giving us Θ(n).
.  Therefore add together to get W(n) = 2W(n/2) + Θ(n)
.  
.  

c) Solve this recurrence using any method you like. Please explain how do you have this.

.  To solve this recurrence, I am going to use the Master Theorem.
.  First, I write the problem in the standard form for solving, a = 2 subproblems, b =2 because (n/b = n/2), the merge or non-recursive work is f(n) = Θ(n).
.  Given these, I compute the pivot term (decides which part of the recurrence dominates). 
.  To do so I look at the depth of the tree, which is approximately logb n. This tells me the number of leaves on the tree is a^(logb a) = n^(logb a).
.  If each leaf does O(1) work, the total leaf work is proportional to n^(logb a).
.  Now I compare f(n) to n^(logb a), finding they are the same order because n^(logb a) = n.
.  This leads me to the Master Theorem Case 2, concluding that W(n) = Θ(n^(logb a) * (log^(k+1) n)) = Θ(nlog n)
.  
.  
.  
.  
.  


d) Assuming that your recursive calls to `num_disagreements_fast` are
done in parallel, write the span recurrence for your algorithm. Please explain how do you have this.

.  For this algorithm, assuming the recursive calls run in parallel, we still must wait for both to complete before sequentially running the combine method.
.  To calculate the span, I began at the top node from which 2 recursive calls run concurrently. Since both are size n/2, the span is S(n/2).
.  After these complete, the sequential merge cost is added of Θ(n).
.  To determine the longest path from start to finish, we must add the max of the 2 recursive calls to the cost of the merge.
.  This ends in the result of S(n) = max{S(n/2), S(n/2)} + Θ(n) = S(n/2) + Θ(n)
.  
.  
.  
.  
.  
.  
.  

e) Solve this recurrence using any method you like. Please explain how do you have this.

.  To solve this recurrence I chose to unroll it and sum.
.  To begin I let Cn upper-bound the merge step...
.  S(n) <= S(n/2) + Cn
.  S(n) <= S(n/4) + C(n/2) + Cn
.  S(n) <= S(n/8) + C(n/4) + C(n/2) + Cn
.  This continues to unravel forever to reach:
.  S(n) <= S(1) + C(n + (n/2) + (n/4) + ... + 1)
.  To sum this geometrically we find S(n) < 2n
.  This results that S(n) <= S(1) + 2Cn = Θ(n)
.  Therefore span is linear
.  
.  

f) If `ranks` is a list of size n, Netflix says it will give you
lg(n) processors to run your algorithm in parallel. What is the
upper bound on the runtime of this parallel implementation? (Hint: assume a Greedy
Scheduler). Please explain how do you have this.

.  The upper bound on the runtime of this parallel implementation is O(n) with lg(n) processors
.  To begin, I found the parallelism (maximum possible speedup) of the algorithm by dividing W/S = Θ((nlog n)/n) = Θ(log n).
.  Given the greedy scheduler we find for P processors, TP <= (W/P) + S
.  Then I plugged in the bounds given P = lgn: 
.  TP <= (W/P) + S = (Θ(nlog n))/(Θ(logn)) + Θ(n) = Θ(n) + Θ(n) = Θ(n)
.  Therefore the upper bound of runtime is Θ(n).
.  
