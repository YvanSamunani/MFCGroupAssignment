# MFCGroupAssignment
### The Algorithm Explained

&nbsp; Let's start by visualizing the World Wide Web as a series of vertices and edges in a directed graph. Each vertex represents a webpage, and each edge represents a link from that page to another. Each edge also carries a weight which is influenced by its origin page's rank.

For this example, imagine the World Wide Web has only five pages, with their links shown below:

<img src="https://s3.amazonaws.com/albertpersonal/graph.png" width="300" height="250"/>

&nbsp; To calculate each page's rank, the weights of each incoming link must be calculated using the other page's rank. But in order to do that, we have to calculate each of the other pages' ranks, then the ranks of the pages with links to those other pages, and so on. When you consider the fact that links can be cyclical, too, this task seems practically impossible!

&nbsp; Fortunately, we do have a very effective and relatively simple way of doing this calculation! This is where linear algebra comes to the rescue. We can create a matrix representation of this graph and use iterative vector multiplication to achieve our result.

Here is the example graph represented as an adjacency list:

`[[1],[4],[0,1,3],[],[1]]`
 Which corresponds to the adjacency matrix below (We will use 0-based indexing for all the matrices):

```
[[0,0,1,0,0],
 [1,0,1,0,1],
 [0,0,0,0,0],
 [0,0,1,0,0],
 [0,1,0,0,0]]
```

&nbsp; Each entry matrix[i][j] represents a link from page j to page i. If the link exists, it is given a value of 1. If not, it is given a value of 0.

From this, we can create the stochastic matrix:

```
[[0, 0, 0.33, 0.2, 0],
 [1, 0, 0.33, 0.2, 1],
 [0, 0, 0.00, 0.2, 0],
 [0, 0, 0.33, 0.2, 0],
 [0, 1, 0.00, 0.2, 0]]
```
&nbsp; What is a stochastic matrix? It is a matrix in which either all the entries of each row or column add up to 1. In this case, the
entries of each column add up to 1, so this matrix is column-stochastic. This type of matrix represents a probability distribution, in which each matrix[i][j] represents the probability that a web surfer who is currently on page j will visit page i.

&nbsp; You may be wondering why column 3 has 0.2 for all its entries, even though Page 3 has no outgoing links! This is because in 
the PageRank algorithm, a page with no outgoing links is assumed to equally distribute its importance among all the existing pages in the network. Effectively, this is the same as saying that the page has links to every page in the network, including itself. The justification for this reasoning is that, using the web surfer model, a user who visits a page with no outbound links will type the url of any of the existing webpages with equal probability. In this example, since there are five pages total in the network, a user on Page 3 will have a 1/5 chance of visiting any of the other pages, hence the 0.2.

&nbsp; To save computational time, I create the stochastic matrix directly from the adjacency list. I also use Python's NumPy library for all matrix operations (You can see the full example code at server/example_pagerank.py). Here is the function in my Python code that creates the stochastic matrix.:

```python
def create_stochastic(adj_list):
  n = len(adj_list)
  matrix = np.zeros((n, n)) #initialize a matrix of zeroes (np is shorthand for NumPy)
  for col, line in enumerate(adj_list):
    if len(line) > 0:
      #case where page has existing links
      for index in line:
        matrix[index][col] = 1/(len(line))
    else:
      #case where page does not have any links
      for index in range(n):
        matrix[index][col] = 1/n
  return matrix
```
