XGBoost is not just a single machine learning algorithm. It combines insights of mathematics and coding designs, such as novel target function and training scheme etc. It is more than a industrial scheme rather than just a algorithm pipeline.

## Prepare Knowledge

1. Calculus (Taylor Expansion)
2. Difference between Decision Tree and Regression Tree(ID3, C4.5, CART)
3. Ensemble algorithms such as Bagging, Boosting
4. Tree algorithms (Random Forest, GBDT, XGBoost, LightGBM)

## Major Insights

- Tree Boosting machine learning algorithm

- Gradient Boosting

- Shrinkage (decaying factor) and Column Subsampling

- Exact Greedy Algorithm(left, right)

- Approximate Algorithm (Weighted Quantile)

- Sparse Data handling

- Parallel Learning (Block storing, Pre-sorting CSC)

## Base Definition

Given a dataset with $N$ examples which contain $m$ dimensional features:
$$
\mathcal{D}=\{(\bold{x}_i,y_i)|i\in[1,N]\cap\mathbb{N},\bold{x}_i\in\mathbb{R}^m,y_i\in\mathbb{R}\}\tag1
$$
Tree ensemble model learns $K$ predictors to combine an ensembled model:
$$
\hat{y}_i=\phi(\bold{x}_i)=\sum_{k=1}^Kf_k(\bold{x}_i)\tag2
$$
where $f_k:\mathbb{R}^m\mapsto\mathbb{R}$ is a regression tree.  Suppose $T_k$ is the number of leaves of the $k^{th}$ regression tree, $\bold{w}_k\in\mathbb{R}^{T_k}$ is the weights of each leaves. Each regression tree maps the input vector to corresponding weights of leaves. The final outputs of the model is the sum of predictions from  each regression tree.

We need to define the target function to control the growing of a regression tree.
$$
J(\phi)=\sum_{i=1}^N{l(y_i,\hat{y_i})+\lambda\sum_{k=1}^K\Omega(f_k)}\tag3
$$
where $l:\mathbb{R}^2\mapsto\mathbb{R}$ is a derivable loss function, and $\Omega(f_k)$ is regularized item to penalize the weights in $f_k$, for avoidance of overfitting.

## Gradient Tree Boosting

#### How to generate a tree

Different from Bagging, which adopts parrallel dataset training, Boosting serializes the training process, the weights of samples differ at each iteration. In iteration $t$, the output is predicted by suming up the previous output and the current iteration optimized regression tree:
$$
\hat{y}_i^{(t)}=\hat{y}_i^{(t-1)}+f_t(\bold{x_i})\tag4
$$
Therefore, in allusion to specific regression tree generation, we revise the target function as:
$$
J^{(t)}=\sum_{i=1}^N{l(y_i,\hat{y}_i^{(t-1)}+f_t(\bold{x}_i))+\lambda\Omega(f_t)}\tag5
$$
Review Taylor Expansion Theory. If $\mathcal{F}\in\mathbb{C}^n[a,b], x_0\in[a,b]$, then $\forall x\in[a,b]$, we have:
$$
\mathcal{F}(x)=\mathcal{F}(x_0)+\sum_{i=1}^{n}\frac{1}{i!}\mathcal{F}^{(i)}(x_0)(x-x_0)^{i}+R_n(x-x_0)\tag6
$$
In our target function, $f_t(\bold{x}_i)$ is variable and $\hat{y}_i^{(t-1)}$ is constant at iteration $t$. In equation (6), we define:
$$
x_0=\hat{y}_i^{(t-1)}
$$

$$
x=\hat{y}_i^{(t-1)}+f_t(\bold{x}_i)
$$

$$
\mathcal{F}(\cdot)=l(y_i,\cdot)
$$

We omit the items after third derivatives in equation (6), and then we get:
$$
J^{(t)}\approx\sum_{i=1}^N[l(y_i,y_i^{(t)})+g_if_t(\bold{x}_i)+\frac{1}{2}h_if_t^2(\bold{x}_i)]+\Omega(f_t)
$$
