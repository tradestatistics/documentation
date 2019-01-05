# The Mathematics of Economic Complexity

This section is adapted from [@atlas2014] but it differs at some points. In particular [@mesuringcomplexity2015] and [@interpretationreflections2014] provide useful technical details.

Our changes here consisted in expressing most of the original equations in terms of matrix and vectors. This is because if you use statistical software such as R, it's more efficient to use linear algebra instead of other operations.

## Revealed Comparative Advantage (RCA)

When associating countries to products it is important to take into account the size of the export volume of countries and that of the world trade of products. This is because, even for the same product, we expect the volume of exports of a large country like China, to be larger than the volume of exports of a small country like Uruguay. By the same, we expect the export volume of products that represent a large fraction of world trade, such as cars or footwear, to represent a larger share of a country's exports than products that account for a small fraction of world trade, like cotton seed oil or potato flour.

To make countries and products comparable we use Balassa's definition of Revealed Comparative Advantage (RCA). Balassa's definition says that a country has revealed Comparative advantage in a product if it exports more than its "fair" share, that is, a share that is equal to the share of total world trade that the product represents. For example, in 2008, with exports of \$42 billion, soybeans represented 0.35% of world trade. Of this total, Brazil exported nearly \$11 billion, and since Brazil's total exports for that year were \$140 billion, soybeans accounted for 7.8% of Brazil's exports. This represents around 21 times Brazil's "fair share" of soybean exports (7.8% divided by 0.35%), so we can say that Brazil has revealed comparative advantage in soybeans.

Let $\hat{x}_{c,p}$ represent the exports of country $c$ in product $p$, just as it was defined in \@ref(data-sources), we can express the Revealed Comparative Advantage that country $c$ has in product $p$ as:

\begin{equation}
\renewcommand{\vec}[1]{\boldsymbol{#1}}
\newcommand{\R}{\mathbb{R}}
(\#eq:rca)
RCA_{c,p} = \frac{\hat{x}_{c,p}}{\sum_c \hat{x}_{c,p}} / \frac{\sum_p \hat{x}_{c,p}}{\sum_{c}\sum_{p} \hat{x}_{c,p}}
\end{equation}

## Smooth Revealed Comparative Advantage (SRCA)

We smoothed changes in export volumes induced by the price fluctuation of commodities by using a modification of \@ref(eq:rca) in which $x_{c,p}$ is averaged over the previous three years by using weights:

$$
SRCA_{c,p}^{(t)} = \frac{\tilde{x}_{c,p}^{(t)}}{\sum_c \tilde{x}_{c,p}^{(t)}} / \frac{\sum_p \tilde{x}_{c,p}^{(t)}}{\sum_{c}\sum_{p} \tilde{x}_{c,p}^{(t)}}
$$

Where

$$
\tilde{x}_{c,p}^{(t)} = \frac{2\hat{x}_{c,p}^{(t)} + \hat{x}_{c,p}^{(t-1)} + \hat{x}_{c,p}^{(t-2)}}{4}
$$

Consider that for some years this needs to be altered. As an example, for 1962 the SRCA is the same as RCA and for 1963 the $x_{c,p}^{(t-3)}$ part is omitted and the denominator is changed to 2.

With this measure we constructed a matrix that connects each country to the products that it makes. The entries in the matrix are 1 if country $c$ exports product $p$ with Revealed Comparative Advantage larger than 1, and 0 otherwise. The matrix $S \in \mathbb{R}^{C\times P}$ has entries defined as:

\begin{equation}
(\#eq:Scp)
s_{c,p} = \begin{cases}1 & \text{ if } SRCA_{c,p}^{(t)} \geq 1\cr 0 & \text{ otherwise}  \end{cases}
\end{equation}

$S$ is the matrix summarizing which country makes what, and is used to construct the product space and our measures of economic complexity for countries and products.

In order to compute some of the equations exposed here we had to reduce $S$ by removing cols and rows where each entry is zero. For some years the number of countries $C$ can be less than 128 as it was exposed in \@ref(data-processing). The number of products $P$ for a given year can also experience a small decrease.

It is also important that beyond computability some products were intensively exported in past decades but then they were replaced by other products. Think of floppy disks exports or saltpeter exports to figure out the dynamic of this matrix over time.

## Diversity and Ubiquity

With $S$ defined as in the previous sections, we can measure Diversity and Ubiquity simply by summing over the rows or columns of that matrix. 

Diversity is defined as:
  
$$k_{c}^{(0)} = \sum_p s_{c,p}$$

And Ubiquity as:
$$k_{p}^{(0)} = \sum_c s_{c,p}$$

## Reflections Method

To generate a more accurate measure of the number of capabilities available in a country, or required by a product, we need to correct the information that diversity and ubiquity carry by using each one to correct the other. For countries, this is to calculate the average ubiquity of the products that it exports, the average diversity of the countries that make those products and so forth. For products, this is to calculate the average diversity of the countries that make them and the average ubiquity of the other products that these countries make. This can be expressed by the recursion:

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
k_{c}^{(n)} = \sum_c \left[\frac{1}{k_{c}^{(0)}} \sum_p s_{c,p} \frac{1}{k_{p}^{(0)}} s_{c,p} \right] k_{c}^{(n-2)}
\end{equation}

The equation above can be conveniently written as a matrix equation (this formulation takes some ideas from [@mesuringcomplexity2015] and [@interpretationreflections2014]):

\begin{equation}
(\#eq:kcn3)
\vec{k}^{(n)} = \hat{S}\vec{k}^{(n-2)}
\end{equation}

Where

\begin{equation}
(\#eq:mcc)
\hat{s}_{c,c'} = \frac{1}{k_{c}^{(0)}} \sum_p s_{c,p} \frac{1}{k_{p}^{(0)}} s_{c,p}
\end{equation}

In a similar way we can define $\tilde{S}$ with entries:

\begin{equation}
(\#eq:mpp)
\tilde{s}_{p,p'} = \frac{1}{k_{p}^{(0)}} \sum_c s_{c,p} \frac{1}{k_{c}^{(0)}} s_{c,p}
\end{equation}

Observe that \@ref(eq:kcn3) is satisfied when $k_{c}^{(n)} = k_{c}^{(n-2)} = 1$. This is the eigenvector of $\tilde{S}$ which is associated with its largest eigenvalue. Since this eigenvector is a vector of ones, it is not informative. We look, instead, for the eigenvector associated with the second largest eigenvalue. This is the eigenvector that captures the largest amount of variance in the system and is our measure of Economic Complexity.

In particular, the interpretation of the scores changes when considering odd or even iteration order $n$, high-order iterations are difficult to interpret, and the process asymptotically converges to a trivial fixed point.

For the analysis we used $n=19$ to compute $k_c$ and $n=20$ to compute $k_p$.

## Economic Complexity Index (ECI)

From the Reflections Method, we define the Economic Complexity Index (ECI) exactly as in [@atlas2014], this is:

\begin{equation}
(\#eq:eci)
ECI_c = \frac{v_c - \mu_{v}}{\sigma_{v}}
\end{equation}

Where

* $\vec{v}$ is a vector whose coordinates are given by $k_{c}^{(19)}$ where $p \in 1,\ldots,C$
* $\mu_v = \sum_c v_c / C$ (mean of $\vec{v}$) 
* $\sigma_v = \sqrt{\sum_c (v_c - \mu_v)^2 / (C - 1)}$ (standard deviation of $\vec{v}$)

## Product Complexity Index (PCI)

Similar to the Economic Complexity Index (ECI), we define a Product Complexity Index (PCI). Because of the symmetry of the problem, this can be done simply by exchanging the index of countries $c$ with that for products $p$ in the definitions above. We define PCI as:

\begin{equation}
(\#eq:pci)
PCI_p = \frac{w_p - \mu_{w}}{\sigma_{w}}
\end{equation}

Where

* $\vec{w}$ is a vector whose coordinates are given by $k_{p}^{(20)}$ where $p \in 1,\ldots,P$
* $\mu_w = \sum_p w_p / P$ (mean of $\vec{w}$)
* $\sigma_w = \sqrt{\sum_p (w_p - \mu_w)^2 / (P - 1)}$ (standard deviation of $\vec{w}$)

## Countries not included in rankings and indicators

The curated data includes all the countries available from UN Comtrade data. However, RCA based calculations such as ECI, PCI, Proximity and Density explained in Chapter \@ref(the-mathematics-of-economic-complexity) consider 128 countries that account for 99% of world trade, 97% of the world’s total GDP and 95% of the world’s population according to [@atlas2014].

We considered simultaneously:

* Countries with population greater or equal to 1.2 million
* Countries whose traded value is greater or equal than 1 billion

<div class="figure">
<img src="fig/countries.svg" alt="Schematic of the procedure used to determine the countries that were included in the Atlas"  />
<p class="caption">(\#fig:unnamed-chunk-1)Schematic of the procedure used to determine the countries that were included in the Atlas</p>
</div>

The full list of included countries is available [here](https://github.com/tradestatistics/atlas-data/blob/master/2-scraped-tables/ranking-1-economic-complexity-index.csv).

## Product Proximity

To make products you need chunks of embedded knowledge which [@atlas2014] calls capabilities. The capabilities needed to produce one good may or may not be useful in the production of other goods. Since we do not observe capabilities directly, we create a measure that infers the similarity between the capabilities required by a pair of goods by looking at the probability that they are coexported. To quantify this similarity we assume that if two goods share most of the requisite capabilities, the countries that export one will also export the other.

Our measure is based on the conditional probability that a country that exports product $p$ will also export product $p'$ (see figure). Since conditional probabilities are not symmetric we take the minimum of the probability of exporting product $p$, given $p'$ and the reverse, to make the measure symmetric and more stringent. As an example, assume that in the year 2017, 20 countries exported X, 10 exported Y and 8 exported both. Then, the product proximity between X and Y is 8/20. 

For a pair of goods $p$ and $p'$ we define Product Proximity $\Phi \in \mathbb{R}^{P\times P}$ as:

$$
\Phi = (S^t S) \odot U
$$

Where $\odot$ denotes element-wise multiplication and 
$$u_{p,p'} = 1 / \max(k_{p}^{(0)}, k_{p'}^{(0)})$$

In other terms, each entry of $\Phi$ corresponds to:
$$
\phi_{p,p'} = \frac{\sum_c s_{c,p} s_{c,p'}}{\max(k_{p}^{(0)}, k_{p'}^{(0)})}
$$

<div class="figure">
<img src="fig/proximity.svg" alt="An illustrative example for the product proximity measure"  />
<p class="caption">(\#fig:unnamed-chunk-2)An illustrative example for the product proximity measure</p>
</div>

## Country Proximity

Similar to \@ref(country-proximity), we define Country Proximity $\Lambda \in \mathbb{R}^{C\times C}$ as:

$$
\Lambda = (SS^t) \odot D
$$

Where 
$$d_{c,c'} = 1 / \max(k_{c}^{(0)}, k_{c'}^{(0)})$$

In other terms, each entry of $\Lambda$ corresponds to:
$$
\lambda_{c,c'} = \frac{\sum_p s_{c,p} s_{c,p'}}{\max(k_{c}^{(0)}, k_{c'}^{(0)})}
$$

## Product Density

Product Density, as introduced in [@productspace2007], is the weighted mean proximity of a new potential product $p$ to a country’s current productive capability. In formal terms this is the matrix $\Psi \in \mathbb{R}^{C\times P}$ defined by:

$$
\Psi = (S\Phi) \oslash (\tilde{S}\Phi)
$$

Where $\tilde{s}_{c,p} = 1$ and $\oslash$ denotes element-wise division.

In other terms, each entry of $\Psi$ corresponds to:
$$
\psi_{c,p} = \frac{\sum_{p} s_{c,p} \phi_{p,p'}}{\sum_{p} \phi_{p,p'}}
$$

## Country Density

Similar to \@ref(product-density), we define Country Density as the matrix $\Omega \in \mathbb{R}^{C\times P}$:

$$
\Omega = (\Lambda S) \oslash (\Lambda\tilde{S})
$$

In other terms, each entry of $\Omega$ corresponds to:
$$
\omega_{c,p} = \frac{\sum_{c} s_{c,p} \phi_{c,c'}}{\sum_{c} \phi_{c,c'}}
$$
