# The Mathematics of Economic Complexity

This section is adapted from [@atlas2014] but it differs at some points. In particular [@measuringcomplexity2015] and [@interpretationreflections2014] provide useful technical details.

Our changes here consisted in expressing most of the original equations in terms of matrix and vectors. This is because if you use statistical software such as R, it's more efficient to use linear algebra instead of other operations.

## Countries not included in rankings and indicators

The curated data includes all the countries available from UN Comtrade data. However, RCA based calculations such as CCI, PCI and Proximity consider 128 countries studied in [@atlas2014] according to a selection process that includes both quality and quantity of data.



The full list of included countries is available [here](https://github.com/tradestatistics/atlas-data/blob/master/2-scraped-tables/ranking-1-economic-complexity-index.csv).

## Revealed Comparative Advantage (RCA)

Let $\renewcommand{\vec}[1]{\boldsymbol{#1}}\newcommand{\R}{\mathbb{R}} X \in \R^{C\times P}$ be a matrix with entries $x_{c,p}$ that represents the exports of country $c$ in product $p$, we can express the Revealed Comparative Advantage that country $c$ has in product $p$ as:

\begin{equation}
(\#eq:Xcp)
((X \oslash (X \vec{1}_{P\times 1}))^t \oslash  (X^t \vec{1}_{C\times 1}  \oslash (\vec{1}_{C\times 1}^t X \vec{1}_{P\times 1})))^t
\end{equation}
$\oslash$ denotes element-wise division and $t$ denotes transposition.

This is the same as:
\begin{equation}
(\#eq:rca)
RCA_{c,p} = \frac{x_{c,p}}{\sum_c x_{c,p}} / \frac{\sum_p x_{c,p}}{\sum_{c}\sum_{p} x_{c,p}}
\end{equation}

## Smooth Revealed Comparative Advantage (SRCA)

Consider the matrix $X$ defined as in \@ref(revealed-comparative-advantage-rca), use the entries from the matrix to create $\hat{X}\in \R^{C\times P}$ with entries

\begin{equation}
(\#eq:Xcp2)
\hat{x}_{c,p}^{(t)} = \frac{2x_{c,p}^{(t)} + x_{c,p}^{(t-1)} + x_{c,p}^{(t-2)}}{4}
\end{equation}

So that $x_{c,p}$ is averaged over the previous three years by using weights. This is done to smooth changes in exports induced by the price fluctuation of products, and instead of equation \@ref(eq:rca) we used

\begin{equation}
(\#eq:srca)
SRCA_{c,p}^{(t)} = \frac{\hat{x}_{c,p}^{(t)}}{\sum_c \hat{x}_{c,p}^{(t)}} / \frac{\sum_p \hat{x}_{c,p}^{(t)}}{\sum_{c}\sum_{p} \hat{x}_{c,p}^{(t)}}
\end{equation}

For some years this needs to be altered. As an example, for 1962 the SRCA is the same as RCA and for 1963 the $x_{c,p}^{(t-3)}$ part is omitted and the denominator is changed to 3.

## Country-product matrix

Taking $\hat{X}$ as with entries as in \@ref(eq:Xcp2) we create $S \in \mathbb{R}^{C\times P}$ with entries defined as:
\begin{equation}
(\#eq:Scp)
s_{c,p} = \begin{cases}1 & \text{ if } SRCA_{c,p}^{(t)} > 1\cr 0 & \text{ otherwise}  \end{cases}
\end{equation}

<!-- $S$ is the matrix summarizing which country makes what, and is used to construct the product space and our measures of economic complexity for countries and products. -->

In order to compute some of the equations exposed here we had to reduce $S$ by removing cols and rows where each entry is zero. For some years the number of countries $C$ can be less than 128 (the number of countries originally included in [@atlas2014]) as it was exposed in \@ref(data-processing). The number of products $P$, which can be 1,222 at maximum (under the HS07 trade classification we are using), can also experience a small decrease for a given year.

## Diversity and Ubiquity

With $S$ defined as in \@ref(eq:Scp), we can measure Diversity and Ubiquity simply by summing over the rows or columns of that matrix. 

Diversity is defined as:
$$\vec{k}_c^0 = S\vec{1}_{P\times 1}$$  

And Ubiquity as:
$$\vec{k}_p^0 = (\vec{1}_{C\times 1}^t S)^t$$

for a single country/product this is the same as
$$k_{c}^{(0)} = \sum_p s_{c,p} \\
k_{p}^{(0)} = \sum_c s_{c,p}$$

## Reflections Method

Economic Complexity is a measure of how much productive knowledge different countries mobilize. This can be computed as a recursion that involves the average ubiquity of the products that a country exports, and the average diversity of the countries that make those products.

Product Complexity is a measure of how much productive knowledge different products mobilize. Its computation is analogous to Economic Complexity, which leads to the recursion:

\begin{equation}
(\#eq:kcn)
k_{c}^{(n)} = \frac{1}{k_{c}^{(0)}} \sum_p s_{c,p} k_{p}^{(n-1)}
\end{equation}

\begin{equation}
(\#eq:kpn)
k_{p}^{(n)} = \frac{1}{k_{p}^{(0)}} \sum_c s_{c,p} k_{c}^{(n-1)}
\end{equation}

Then we insert \@ref(eq:kpn) into \@ref(eq:kcn) to obtain:

\begin{equation}
(\#eq:kcn2)
k_{c}^{(n)} = \sum_c \left(\frac{1}{k_{c}^{(0)}} \sum_p s_{c,p} s_{c,p} \frac{1}{k_{p}^{(0)}} s_{c',p}\right) k_{c}^{(n-2)}
\end{equation}

\@ref(eq:kcn2) above can be conveniently written as a matrix equation (this formulation takes some ideas from [@measuringcomplexity2015] and [@interpretationreflections2014]):

\begin{equation}
(\#eq:kcn3)
\vec{k}_c^{(n)} = \hat{S}\vec{k}_c^{(n-2)} \\
\vec{k}_p^{(n)} = \tilde{S}\vec{k}_p^{(n-2)}
\end{equation}

Where
\begin{equation}
(\#eq:mcc)
\hat{S} = (S \oslash K_c^{(0)}) (S^t \oslash K_p^{(0)})
\end{equation}
and
\begin{equation}
(\#eq:mpp)
\tilde{S} = (S^t \oslash D) (S \oslash U)
\end{equation}
$\oslash$ denotes element-wise division, $d_{c,p} = k_c^{(0)}$ and $u_{p,c} = k_p^{(0)}$.

In particular, the interpretation of the scores changes when considering odd or even iteration order $n$, high-order iterations are difficult to interpret, and the process asymptotically converges to a trivial fixed point according to [@measuringcomplexity2015].

For the analysis we used $n=19$ to compute $k_c$ and $n=20$ to compute $k_p$.

## Country Complexity Index (CCI)

From the \@ref(reflections-method), we define the Economic Complexity Index (CCI) exactly as in [@atlas2014], this is:

\begin{equation}
(\#eq:cci)
CCI_c = \frac{v_c - \mu_{v}}{\sigma_{v}}
\end{equation}

Where

* $\vec{v}$ is a vector whose coordinates are given by $k_{c}^{(19)}$ where $c \in 1,\ldots,C$
* $\mu_v = \sum_c v_c / C$ (mean of $\vec{v}$) 
* $\sigma_v = \sqrt{\sum_c (v_c - \mu_v)^2 / (C - 1)}$ (standard deviation of $\vec{v}$)

## Product Complexity Index (PCI)

Similar to the Economic Complexity Index (CCI), we define a Product Complexity Index (PCI) from \@ref(reflections-method). We define PCI as:

\begin{equation}
(\#eq:pci)
PCI_p = \frac{w_p - \mu_{w}}{\sigma_{w}}
\end{equation}

Where

* $\vec{w}$ is a vector whose coordinates are given by $k_{p}^{(20)}$ where $p \in 1,\ldots,P$
* $\mu_w = \sum_p w_p / P$ (mean of $\vec{w}$)
* $\sigma_w = \sqrt{\sum_p (w_p - \mu_w)^2 / (P - 1)}$ (standard deviation of $\vec{w}$)

## Eigenvalues method

An alternative to obtain the Economic Complexity Index from \@ref(economic-complexity-index-cci) is to take \@ref(eq:Scp) and compute the eigenvector associated to the second largest eigenvalue of the matrix:
$$(S \oslash D) (S^t \oslash U)$$
Where $\oslash$ denotes element-wise division, $d_{c,p} = k_c^{(0)}$ and $u_{p,c} = k_p^{(0)}$.

Analogously, the Product Complexity Index from \@ref(product-complexity-index-pci) corresponds to the associated eigenvector of the second largest eigenvalue of the matrix:
$$(S^t \oslash U) (S \oslash D)$$

The resulting vectors are standardized by taking
$$CCI_c = \frac{v_c - \mu_{v}}{\sigma_{v}}\:, PCI_p = \frac{w_p - \mu_{w}}{\sigma_{w}}$$
Where $\mu_x$ denotes the mean of $x$ and $\sigma_x$ denotes de standard deviation of $x$.

## Fitness method

Another alternative to obtain the Economic Complexity Index from \@ref(economic-complexity-index-cci) and Product Complexity Index from \@ref(product-complexity-index-pci) is to start with $\vec{k}_{c}^{(0)} = \vec{1}$ and $\vec{k}_{p}^{(0)} = \vec{1}$. From that initial condition the next steps are to compute:

\begin{equation}
\vec{k}_c^{(n)} = S \hat{\vec{k}}_p^{(n-1)}\\
\vec{k}_p^{(n)} = S^t \vec{\hat{k}}_c^{(n-1)}
\end{equation}

where
\begin{equation}
\hat{k}_c^{(n-1)} = (k_c^{(n-1)})^{-\gamma}\\
\hat{k}_p^{(n-1)} = (k_p^{(n-1)})^{-1/\gamma}
\end{equation}
for $\gamma > 0$ (the original version of the Fitness Method uses $\gamma = 1$).

In addition, each step of the method involves to divide the result of $\vec{k}_c^{(n)}$ and $\vec{k}_p^{(n)}$ by its mean, so that the final equations for each step are:
\begin{equation}
\tilde{\vec{k}}_c^{(n)} = \vec{k}_c^{(n)} / \mu_{\vec{k}_c^{(n)}}\\
\tilde{\vec{k}}_p^{(n)} = \vec{k}_p^{(n)} / \mu_{\vec{k}_p^{(n)}}
\end{equation}

The resulting vectors are standardized by taking
$$CCI_c = \frac{v_c - \mu_{v}}{\sigma_{v}},\: PCI_p = \frac{w_p - \mu_{w}}{\sigma_{w}}$$

## Product Proximity

For a pair of goods $p$ and $p'$ we define Product Proximity $\Phi \in \mathbb{R}^{P\times P}$ as:

$$
\Phi = (S^t S) \oslash U
$$
where $\oslash$ denotes element-wise division and $u_{p,p'} = \max(k_{p}^{(0)}, k_{p'}^{(0)})$.

In other terms, each entry of $\Phi$ corresponds to:
$$
\phi_{p,p'} = \frac{\sum_c s_{c,p} s_{c,p'}}{\max(k_{p}^{(0)}, k_{p'}^{(0)})}
$$



## Country Proximity

Similar to \@ref(country-proximity), we define Country Proximity $\Lambda \in \mathbb{R}^{C\times C}$ as:

$$
\Lambda = (SS^t) \oslash D
$$
where $\oslash$ denotes element-wise division and $d_{p,p'} = \max(k_{c}^{(0)}, k_{c'}^{(0)})$.

In other terms, each entry of $\Lambda$ corresponds to:
$$
\lambda_{c,c'} = \frac{\sum_p s_{c,p} s_{c,p'}}{\max(k_{c}^{(0)}, k_{c'}^{(0)})}
$$
