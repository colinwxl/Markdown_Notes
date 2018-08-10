## An illustrated introduction to the t-SNE algorithm
[Original Site](https://www.oreilly.com/learning/an-illustrated-introduction-to-the-t-sne-algorithm):  t-distributed stochastic neighbor embedding

How can we possibly reduce the dimensionality of a dataset from an arbitrary number to two or three, which is what we’re doing when we visualize data on a screen?

The answer lies in the observation that many real-world datasets have <font color=red>a low intrinsic dimensionality</font>, even though they’re embedded in a high-dimensional space. Imagine that you’re shooting a panoramic landscape with your camera, while rotating around yourself. We can consider every picture as a point in a 16,000,000-dimensional space (assuming a 16 megapixels camera). Yet, the set of pictures approximately lie in a three-dimensional space (yaw, pitch, roll). This low-dimensional space is embedded within the high-dimensional space in a complex, nonlinear way. Hidden in the data, this structure can only be recovered via specific mathematical methods.

### Implementation:
```python
import numpy as np
from sklearn.manifold import TSNE
from sklearn.datasets import load_digits
# Random state.
RS = 20150101
import matplotlib.pyplot as plt
import matplotlib.patheffects as PathEffects
import matplotlib
%matplotlib inline

import seaborn as sns
sns.set_style('darkgrid')
sns.set_palette('muted')
sns.set_context("notebook", font_scale=1.5, rc={"lines.linewidth": 2.5})


# Now we load the classic handwritten digits datasets. It contains 1797 images with 8∗8=64 pixels each.
digits = load_digits()
digits.data.shape
# print(digits['DESCR']) to see the dataset's description

nrows, ncols = 2, 5
plt.figure(figsize=(6,3))
plt.gray()
for i in range(ncols * nrows):
    ax = plt.subplot(nrows, ncols, i + 1)
    ax.matshow(digits.images[i,...])
    plt.xticks([]); plt.yticks([])
    plt.title(digits.target[i])
plt.savefig('digits-generated.png', dpi=150)

# We first reorder the data points according to the handwritten numbers.
X = np.vstack([digits.data[digits.target==i] for i in range(10)])
y = np.hstack([digits.target[digits.target==i] for i in range(10)])

digits_proj = TSNE(random_state=RS).fit_transform(X)

# Here is a utility function used to display the transformed dataset.
# The color of each point refers to the actual digit (of course, 
# this information was not used by the dimensionality reduction algorithm).

def scatter(x, colors):
    # We choose a color palette with seaborn.
    palette = np.array(sns.color_palette("hls", 10))

    # We create a scatter plot.
    f = plt.figure(figsize=(8, 8))
    ax = plt.subplot(aspect='equal')
    sc = ax.scatter(x[:,0], x[:,1], lw=0, s=40,
                    c=palette[colors.astype(np.int)])
    plt.xlim(-25, 25)
    plt.ylim(-25, 25)
    ax.axis('off')
    ax.axis('tight')

    # We add the labels for each digit.
    txts = []
    for i in range(10):
        # Position of each label.
        xtext, ytext = np.median(x[colors == i, :], axis=0)
        txt = ax.text(xtext, ytext, str(i), fontsize=24)
        txt.set_path_effects([
            PathEffects.Stroke(linewidth=5, foreground="w"),
            PathEffects.Normal()])
        txts.append(txt)

    return f, ax, sc, txts

scatter(digits_proj, y)
plt.savefig('digits_tsne-generated.png', dpi=120)
```

### The Mathematics
A **data point** is a point $ x_i $ in the original **data space** $R^D$, where $D=64$ is the **dimensionality** of the data space. Every point is an image of a handwritten digit here. There are $N=1797$ points.

A **map point** is a point $y_i$ in the **map space** $R^2$. This space will contain our final representation of the dataset. There is a bijection between the data points and the map points: every map point represents one of the original images.

How do we choose the positions of the map points? We want to conserve the structure of the data. More specifically, if two data points are close together, we want the two corresponding map points to be close too. Let’s $|x_i-x_j|$ be the Euclidean distance between two data points, and $|y_i-y_j|$ the distance between the map points. We first define a conditional similarity between the two data points:
$$p_{j|i} = \frac{ {\rm exp}\bigg\lgroup -|x_i-x_j|^2\bigg/2\sigma_i^2\bigg\rgroup}{\sum\limits_{k\neq i}{\rm exp}\bigg\lgroup -|x_i-x_k|^2\bigg/2\sigma_i^2 \ \bigg\rgroup}$$

This measures how close $x_j$ is from $x_i$, considering a **Gaussian distribution** around $x_i$ with a given variance $\sigma_i^2$. This variance is different for every point; it is chosen such that points in dense areas are given a smaller variance than points in sparse areas. The original paper details how this variance is computed exactly.

Now, we define the similarity as a symmetrized version of the conditional similarity:
$p_{ij} = \frac{p_{j|i}+p_{i|j}}{2N}$

We obtain a similarity matrix for our original dataset. What does this matrix look like?

**Similarity matrix**
The following function computes the similarity with a constant $\sigma$.
```python
def _joint_probabilities_constant_sigma(D, sigma):
    P = np.exp(-D**2/(2 * sigma**2))
    P /= np.sum(P, axis=1)
    return P
```
We now compute the similarity with a $σ_i$ depending on the data point (found via a binary search, according to the original t-SNE paper). This algorithm is implemented in the `_joint_probabilities` private function in scikit-learn’s code.

```python
from scipy.spatial.distance import squareform, pdist
from sklearn.metrics.pairwise import pairwise_distances
from sklearn.manifold.t_sne import (_joint_probabilities, _kl_divergence)
from sklearn.utils.extmath import _ravel

# Pairwise distances between all data points.
D = pairwise_distances(X, squared=True)
# Similarity with constant sigma.
P_constant = _joint_probabilities_constant_sigma(D, .002)
# Similarity with variable sigma.
P_binary = _joint_probabilities(D, 30., False)
# The output of this function needs to be reshaped to a square matrix.
P_binary_s = squareform(P_binary)
```
We can now display the distance matrix of the data points, and the similarity matrix with both a constant and variable sigma.

```python
plt.figure(figsize=(12, 4))
pal = sns.light_palette("blue", as_cmap=True)

plt.subplot(131)
plt.imshow(D[::10, ::10], interpolation='none', cmap=pal)
plt.axis('off')
plt.title("Distance matrix", fontdict={'fontsize': 16})

plt.subplot(132)
plt.imshow(P_constant[::10, ::10], interpolation='none', cmap=pal)
plt.axis('off')
plt.title("$p_{j|i}$ (constant $\sigma$)", fontdict={'fontsize': 16})

plt.subplot(133)
plt.imshow(P_binary_s[::10, ::10], interpolation='none', cmap=pal)
plt.axis('off')
plt.title("$p_{j|i}$ (variable $\sigma$)", fontdict={'fontsize': 16})
plt.savefig('similarity-generated.png', dpi=120)
```
We can already observe the 10 groups in the data, corresponding to the 10 numbers.

Let’s also define a similarity matrix for our map points.

$$q_{ij}=\frac{f(|x_i-x_j|)}{\sum\limits_{k\neq i}f(|x_i-x_k|)}\ with\ f(z) =\frac{1}{1+z^2}$$

This is the same idea as for the data points, but with a different distribution (**t-Student with one degree of freedom**, or **Cauchy distribution**, instead of a Gaussian distribution). We’ll elaborate on this choice later.

Whereas the data similarity matrix ($p_{ij}$) is fixed, the map similarity matrix ($q_{ij}$) depends on the map points. What we want is for these two matrices to be as close as possible. This would mean that similar data points yield similar map points.

### A physical analogy 
Let’s assume that our map points are all connected with springs (弹簧). The stiffness (僵硬) of a spring connecting points i and j depends on the mismatch between the similarity of the two data points and the similarity of the two map points, that is, $p_{ij}-q_{ij}$. Now, we let the system evolve according to the laws of physics. If two map points are far apart while the data points are close, they are attracted together. If they are nearby while the data points are dissimilar, they are repelled.

The final mapping is obtained when the equilibrium is reached.

Here is an illustration of a dynamic graph layout based on a similar idea. Nodes are connected via springs and the system evolves according to law of physics (example by [Mike Bostock](https://bl.ocks.org/mbostock/4062045)).

<img src="https://t1.picb.cc/uploads/2018/08/09/Juc8Ti.jpg" style="width:400px;height:450px;"></img>

### Algorithm
Remarkably, this physical analogy stems naturally from the mathematical algorithm. It corresponds to minimizing the Kullback-Leiber divergence (KL距离、相对熵，在相同事件空间里，概率分布$P(x)$的事件空间，若用概率分布$Q(x)$编码时，平均每个基本事件（符号）编码长度增加了多少比特) between the two distributions ($p_{ij}$) and ($q_{ij}$):

$$KL(P||Q)=\sum\nolimits_{i,j}p_{ij}{\rm log}\frac{p_{ij}}{q_{ij}}$$.

This measures the distance between our two similarity matrices.

To minimize this score, we perform a gradient descent. The gradient can be computed analytically:

$$\frac{\partial KL(P||Q)}{\partial y_i}=4\sum\nolimits_j(p_{ij}-q_{ij})g(|x_i-x_j|)u_{ij}\ with\ g(z)=\frac{z}{1+z^2}$$

Here, $u_{ij}$ is a unit vector going from $y_j$ to $y_i$. This gradient expresses the sum of all spring forces applied to map point $i$.

Let’s illustrate this process by creating an animation of the convergence. We’ll have to monkey patch the internal `_gradient_descent()` function from scikit-learn’s t-SNE implementation in order to register the position of the map points at every iteration.
```python
# This list will contain the positions of the map points at every iteration.
positions = []
def _gradient_descent(objective, p0, it, n_iter, objective_error=None,
                      n_iter_check=1, n_iter_without_progress=50,
                      momentum=0.5, learning_rate=1000.0, min_gain=0.01,
                      min_grad_norm=1e-7, min_error_diff=1e-7, verbose=0,
                      args=None, kwargs=None):
    """Batch gradient descent with momentum and individual gains.

    Parameters
    ----------
    objective : function or callable
        Should return a tuple of cost and gradient for a given parameter
        vector. When expensive to compute, the cost can optionally
        be None and can be computed every n_iter_check steps using
        the objective_error function.

    p0 : array-like, shape (n_params,)
        Initial parameter vector.

    it : int
        Current number of iterations (this function will be called more than
        once during the optimization).

    n_iter : int
        Maximum number of gradient descent iterations.

    n_iter_check : int
        Number of iterations before evaluating the global error. If the error
        is sufficiently low, we abort the optimization.

    objective_error : function or callable
        Should return a tuple of cost and gradient for a given parameter
        vector.

    n_iter_without_progress : int, optional (default: 30)
        Maximum number of iterations without progress before we abort the
        optimization.

    momentum : float, within (0.0, 1.0), optional (default: 0.5)
        The momentum generates a weight for previous gradients that decays
        exponentially.

    learning_rate : float, optional (default: 1000.0)
        The learning rate should be extremely high for t-SNE! Values in the
        range [100.0, 1000.0] are common.

    min_gain : float, optional (default: 0.01)
        Minimum individual gain for each parameter.

    min_grad_norm : float, optional (default: 1e-7)
        If the gradient norm is below this threshold, the optimization will
        be aborted.

    min_error_diff : float, optional (default: 1e-7)
        If the absolute difference of two successive cost function values
        is below this threshold, the optimization will be aborted.

    verbose : int, optional (default: 0)
        Verbosity level.

    args : sequence
        Arguments to pass to objective function.

    kwargs : dict
        Keyword arguments to pass to objective function.

    Returns
    -------
    p : array, shape (n_params,)
        Optimum parameters.

    error : float
        Optimum.

    i : int
        Last iteration.
    """
    if args is None:
        args = []
    if kwargs is None:
        kwargs = {}

    p = p0.copy().ravel()
    update = np.zeros_like(p)
    gains = np.ones_like(p)
    error = np.finfo(np.float).max
    best_error = np.finfo(np.float).max
    best_iter = 0

    for i in range(it, n_iter):
        positions.append(p.copy())
        new_error, grad = objective(p, *args, **kwargs)
        grad_norm = linalg.norm(grad)

        inc = update * grad >= 0.0
        dec = np.invert(inc)
        gains[inc] += 0.05
        gains[dec] *= 0.95
        np.clip(gains, min_gain, np.inf)
        grad *= gains
        update = momentum * update - learning_rate * grad
        p += update

        if (i + 1) % n_iter_check == 0:
            if new_error is None:
                new_error = objective_error(p, *args)
            error_diff = np.abs(new_error - error)
            error = new_error

            if verbose >= 2:
                m = "[t-SNE] Iteration %d: error = %.7f, gradient norm = %.7f"
                print(m % (i + 1, error, grad_norm))

            if error < best_error:
                best_error = error
                best_iter = i
            elif i - best_iter > n_iter_without_progress:
                if verbose >= 2:
                    print("[t-SNE] Iteration %d: did not make any progress "
                          "during the last %d episodes. Finished."
                          % (i + 1, n_iter_without_progress))
                break
            if grad_norm <= min_grad_norm:
                if verbose >= 2:
                    print("[t-SNE] Iteration %d: gradient norm %f. Finished."
                          % (i + 1, grad_norm))
                break
            if error_diff <= min_error_diff:
                if verbose >= 2:
                    m = "[t-SNE] Iteration %d: error difference %f. Finished."
                    print(m % (i + 1, error_diff))
                break

        if new_error is not None:
            error = new_error

    return p, error, i
sklearn.manifold.t_sne._gradient_descent = _gradient_descent
```
Let’s run the algorithm again, but this time saving all intermediate positions.
```python
X_proj = TSNE(random_state=RS).fit_transform(X)
X_iter = np.dstack(position.reshape(-1, 2) for position in positions)
# We create an animation using MoviePy.
f, ax, sc, txts = scatter(X_iter[..., -1], y)

def make_frame_mpl(t):
    i = int(t*40)
    x = X_iter[..., i]
    sc.set_offsets(x)
    for j, txt in zip(range(10), txts):
        xtext, ytext = np.median(x[y == j, :], axis=0)
        txt.set_x(xtext)
        txt.set_y(ytext)
    return mplfig_to_npimage(f)

animation = mpy.VideoClip(make_frame_mpl,
                          duration=X_iter.shape[2]/40.)
animation.write_gif("animation1.gif", fps=20)
```
<img src="https://t1.picb.cc/uploads/2018/08/09/JucSXN.gif" style="width:400px;height:450px;"></img>

We can clearly observe the different phases of the optimization, as described in the original paper.

Let’s also create an animation of the similarity matrix of the map points. We’ll observe that it’s getting closer and closer to the similarity matrix of the data points.

```python
n = 1. / (pdist(X_iter[..., -1], "sqeuclidean") + 1)
Q = n / (2.0 * np.sum(n))
Q = squareform(Q)

f = plt.figure(figsize=(6, 6))
ax = plt.subplot(aspect='equal')
im = ax.imshow(Q, interpolation='none', cmap=pal)
plt.axis('tight')
plt.axis('off')

def make_frame_mpl(t):
    i = int(t*40)
    n = 1. / (pdist(X_iter[..., i], "sqeuclidean") + 1)
    Q = n / (2.0 * np.sum(n))
    Q = squareform(Q)
    im.set_data(Q)
    return mplfig_to_npimage(f)

animation = mpy.VideoClip(make_frame_mpl,
                          duration=X_iter.shape[2]/40.)
animation.write_gif("animation2.gif", fps=20)
```
![](https://d3ansictanv2wj.cloudfront.net/images/animation_matrix-da2d5f1b.gif)

### The t-Student distribution
Let’s now explain the choice of the t-Student distribution for the map points, while a normal distribution is used for the data points. It is well known that the volume of the $N$-dimensional ball of radius $r$ scales as $r^N$. When $N$ is large, if we pick random points uniformly in the ball, most points will be close to the surface, and very few will be near the center.

This is illustrated by the following simulation, showing the distribution of the distances of these points, for different dimensions.

```python
npoints = 1000
plt.figure(figsize=(15, 4))
for i, D in enumerate((2, 5, 10)):
    # Normally distributed points.
    u = np.random.randn(npoints, D)
    # Now on the sphere.
    u /= norm(u, axis=1)[:, None]
    # Uniform radius.
    r = np.random.rand(npoints, 1)
    # Uniformly within the ball.
    points = u * r**(1./D)
    # Plot.
    ax = plt.subplot(1, 3, i+1)
    ax.set_xlabel('Ball radius')
    if i == 0:
        ax.set_ylabel('Distance from origin')
    ax.hist(norm(points, axis=1),
            bins=np.linspace(0., 1., 50))
    ax.set_title('D=%d' % D, loc='left')
plt.savefig('spheres-generated.png', dpi=100, bbox_inches='tight')
```
![](https://d3ansictanv2wj.cloudfront.net/images/spheres-eaa90761.png)

When reducing the dimensionality of a dataset, if we used the same Gaussian distribution for the data points and the map points, we would get an imbalance in the distribution of the distances of a point’s neighbors. This is because the distribution of the distances is so different between a high-dimensional space and a low-dimensional space. Yet, the algorithm tries to reproduce the same distances in the two spaces. This imbalance would lead to an excess of attraction forces and a sometimes unappealing mapping. This is actually what happens in the original SNE algorithm, by [Hinton and Roweis (2002)](http://www.cs.toronto.edu/~fritz/absps/sne.pdf).

The t-SNE algorithm works around this problem by using a t-Student with one degree of freedom (or Cauchy) distribution for the map points. This distribution has a **much heavier tail** than the Gaussian distribution, which compensates the original imbalance. For a given similarity between two data points, the two corresponding map points will need to be much further apart in order for their similarity to match the data similarity. This can be seen in the following plot.

```python
z = np.linspace(0., 5., 1000)
gauss = np.exp(-z**2)
cauchy = 1/(1+z**2)
plt.plot(z, gauss, label='Gaussian distribution')
plt.plot(z, cauchy, label='Cauchy distribution')
plt.legend()
plt.savefig('distributions-generated.png', dpi=100)
```
![](https://d3ansictanv2wj.cloudfront.net/images/distributions-a5228a2e.png)
Using this distribution leads to more effective data visualizations, where clusters of points are more distinctly separated.

### Conclusion
The t-SNE algorithm provides an effective method to visualize a complex dataset. It successfully uncovers hidden structures in the data, exposing natural clusters and smooth nonlinear variations along the dimensions. It has been implemented in many languages, including Python, and it can be easily used thanks to the scikit-learn library.

The references below describe some optimizations and improvements that can be made to the algorithm and implementations. In particular, the algorithm described here is quadratic(二次的) in the number of samples, which makes it unscalable to large datasets. One could for example obtain an $O(NlogN)$ complexity by using the Barnes-Hut algorithm to accelerate the N-body simulation via a quadtree(四叉树) or an octree(八叉树).

### References
- [Original paper](http://jmlr.csail.mit.edu/papers/volume9/vandermaaten08a/vandermaaten08a.pdf)
- [Optimized t-SNE paper](http://lvdmaaten.github.io/publications/papers/JMLR_2014.pdf)
- [A notebook on t-SNE by Alexander Flabish](http://nbviewer.ipython.org/urls/gist.githubusercontent.com/AlexanderFabisch/1a0c648de22eff4a2a3e/raw/59d5bc5ed8f8bfd9ff1f7faa749d1b095aa97d5a/t-SNE.ipynb)
- [Official t-SNE page](http://lvdmaaten.github.io/tsne/)
- [scikit documentation](http://scikit-learn.org/stable/modules/generated/sklearn.manifold.TSNE.html)
- [Barnes-Hut t-SNE implementation in Python](https://github.com/danielfrg/tsne)
- [Barnes-Hut on Wikipedia](http://en.wikipedia.org/wiki/Barnes%E2%80%93Hut_simulation)
- [t-SNE on Wikipedia](http://en.wikipedia.org/wiki/T-distributed_stochastic_neighbor_embedding)
- [Implementation in scikit-learn](https://github.com/scikit-learn/scikit-learn/blob/master/sklearn/manifold/t_sne.py)


### Appendix (Some practical examples)
#### 1. A simple example: the Iris dataset
t-SNE can help us to decide whether classes are separable in some linear or nonlinear representation. 
```python
from sklearn.manifold import TSNE
from sklearn.datasets import load_iris
from sklearn.decomposition import PCA
import matplotlib.pyplot as plt

iris = load_iris()
X_tsne = TSNE(learning_rate=100).fit_transform(iris.data)
X_pca = PCA().fit_transform(iris.data)

plt.figure(figsize=(10, 5))
plt.subplot(121)
plt.scatter(X_tsne[:, 0], X_tsne[:, 1], c=iris.target)
plt.subplot(122)
plt.scatter(X_pca[:, 0], X_pca[:, 1], c=iris.target)
```

#### 2. High-dimensional sparse data: the 20 newsgroups dataset
```python
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt
from sklearn.datasets import fetch_20newsgroups
from sklearn.feature_extraction.text import TfidfVectorizer

categories = ['alt.atheism', 'talk.religion.misc', 'comp.graphics', 'sci.space']
newsgroups = fetch_20newsgroups(subset="train", categories=categories)
vectors = TfidfVectorizer().fit_transform(newsgroups.data)
# print(repr(vectors))

# For high-dimensional sparse data it is helpful to first reduce the dimensions to 50 dimensions with TruncatedSVD and then perform t-SNE. This will usually improve the visualization.
from sklearn.decomposition import TruncatedSVD
X_reduced = TruncatedSVD(n_components=50, random_state=0).fit_transform(vectors)

X_embedded = TSNE(n_components=2, perplexity=40, verbose=2).fit_transform(X_reduced)

fig = plt.figure(figsize=(10, 10))
ax = plt.axes(frameon=False)
plt.setp(ax, xticks=(), yticks=())
plt.subplots_adjust(left=0.0, bottom=0.0, right=1.0, top=0.9, wspace=0.0, hspace=0.0)
plt.scatter(X_embedded[:, 0], X_embedded[:, 1], c=newsgroups.target, marker="x")
```


