# MDistil

-----------

This is the documentation to the implementation of Distillation [Peardon et al., 2009, 0905.2160] and stochastic LapH [Morningstar et al., 2011, 1104.3870] in grid.

## LapEvec

### Template structure

One template argument `FImpl`, expected to be a fermion implementation.

### Description

Calculates the lowest $ n_\mathrm{vec} $ `Laplacian eigenvectors` $v$ and `eigenvalues` $\lambda$ by solving the Laplacian eigenvalue equation

$$\sum_{b,\vec{y}}-\nabla^2_{ab}(\vec{x},\vec{y};t) v_{kb}(\vec{y};t) =  \lambda_k(t) v_{ka}(\vec{x};t)$$

on each timeslice $ t $ with the 3-dimensional lattice `Laplacian` 

$$-\nabla^2_{ab}(\vec{x},\vec{y};t) = 6 \delta_{xy} - \sum_{j=1}^3 (\tilde{U}^{ab}_j(\vec{x},t) \delta_{x+\hat{j},y} + \tilde{U}^{ba}_j(\vec{y},t)^* \delta_{x-\hat{j},y} )$$

where \tilde{U} are Stout-smeared [Morningstar, Peardon: Phys.Rev. D69 (2004) 054501] gauge links using $ n_\mathrm{step} $ smearing steps with a weight of $ \rho $ (only on the spatial directions).

These eigenvectors define the hermitian smearing matrix

$$ S_{xy}(t) = \sum_{k=1}^{N_\mathrm{vec}} v_{ka}(\vec{x};t) v_{ka}^\dagger(\vec{y};t) $$

### Parameters

| Parameter             | Type                       | Description            |
|-----------------------|----------------------------|------------------------|
| `gauge field`         | `FermionField`             | $U_{\mu}$              |
| `StoutParameters`     | `{int,double}`             | steps, parm            |
| `ChebyshevParameters` | `{int,double,double}`      | PolyOrder, alpha, beta |
| `LanczosParameters`   | `{int,int,int,int,double}` | Nvec,Nk,Np,MaxIt,resid |

### Dependencies

- gauge field

### Products

Laplacian eigenvectors, named `eigenPack` (Note: The eigenvectors are 4d-eigenvectors, which are reinterpreted from the 3d-Lapalcian eigenvectors. The eigenvalues of the eigenpack are those of $t=0$.)

-----------

## Perambulator

### Template structure

One template argument `FImpl`, expected to be a fermion implementation.

### Description

Computes the `(stochastic) perambulator` $\tau$ from the laplacian eigenevctors

$$ \tau_{k\alpha}^{[n,d]}(t') = \sum_{a,b,\beta,t,\vec{x}',\vec{x}} v_{ka}(\vec{x}';t')^\dagger D^{-1}_{a\alpha,b\beta}(\vec{x}',t';\vec{x},t) \varrho^{[n,d]}_{b \beta} (\vec{x},t) $$

defined via the quark source vectors

$$  \varrho^{[n,d]}_{a \alpha} (\vec{x},t)  = \sum_{k,l,t',\beta} v_{ka}(\vec{x};t) P_{k\alpha,l\beta}^{[d]}(t,t') \rho_{l \beta}^{[n]}(t') \, , $$

which are constructed from the Laplacian eigenevectors from the LapEvec module, $N_\mathrm{noise}$ noise vectors

$$ \rho_{k\alpha}^{[n]}(t)\, , $$

which have the expectation value

$$ \langle \rho^{[n_1]} \rho^{[n_2]\dagger} \rangle = \delta^{n_1, n_2} \, ,$$

and $N_\mathrm{dil}$ dilution projectors

$$ P_{k\alpha,l\beta}^{[d]}(t,t') $$

which have the properties

$$ (P^{[d]})^2 = P^{[d]} \ , \ \sum_d P^{[d]} = 1 $$

The solver parameters (mass,M5,CGPrecision,MaxIter) have to be supplied for the inversion of the Dirac matrix. The noises are generated by a unique string which is compound of the gauge field used and the mass given to the solver. (STILL TO BE DONE AND DISCUSSED!)

The perambulators can in turn be used to define the quark sink vectors

$$ \varphi^{[n,d]}_{a \alpha} (\vec{x},t)  = \sum_{k} v_{ka}(\vec{x};t) \tau_{k\alpha}^{[n,d]}(t)  $$

using which a smeared-to-smeared quark propagator can be expressed via

$$ \langle \tilde{q}_{a \alpha} (\vec{x}',t') \bar{\tilde{q}}_{b \beta} (\vec{x},t) \rangle = \bigg\langle \sum_d \varphi^{[n,d]}_{a \alpha} (\vec{x},t) \varrho^{[n,d]}_{b \beta} (\vec{x},t) \bigg\rangle $$

The code in grid allows for inderlaced dilution in spin (interlace-$I_\alpha$), time (interlace-$I_t$) and laplacian eigenmodes (interlace-$I_k$) - the dilution index $d$ is therefore implemented as a compound index, $d = (d_\alpha,d_t,d_k)$. 

For $I_t=N_t$, $I_\alpha=N_s$, $I_k=N_\mathrm{vec}$ we reproduce the exact `perambulator` $\tau$ 

$$ \tau_{k\alpha, l\beta}(t',t) = \sum_{a,b,\vec{x},\vec{x}'} v_{ka}(\vec{x}';t')^\dagger D^{-1}_{a\alpha,b\beta}(\vec{x}',t';\vec{x},t) v_{lb}(\vec{x};t) \, . $$

In this case of exact distillation, the noise vectors are automatically replaced by vectors of ones, but $N_\mathrm{noise}$ has to be $=1$ ior the code will throw an error.


### Parameters

| Parameter          | Type                         | Description                            |
|--------------------|------------------------------|----------------------------------------|
| `eigenvectors`     | `eigenPack`                  | $v_k$                                  |
| `SolverParameters` | `{double,int,double,double}` | CGPrecision,MaxIterations,mass,M5      |
| `DistilParameters` | `std::vector<int>`           | TI,LI,SI,Nt,Nvec,Ns,nnoise,tsrc,Nt_inv |

### Dependencies

- Laplacian eigenPack


### Products

- `perambulator` $\tau$

- `noises` $\rho$

-----------


## DistilVecs

### Template structure

One template argument `FImpl`, expected to be a fermion implementation.

### Description

Calculates the quark source vectors

$$  \varrho^{[n,d]}_{a \alpha} (\vec{x},t)  = \sum_{k,l,t',\beta} v_{ka}(\vec{x};t) P_{k\alpha,l\beta}^{[d]}(t,t') \rho_{l \beta}^{[n]}(t') $$

from the eigenvectors and noises and the quark sink vectors

$$ \varphi^{[n,d]}_{a \alpha} (\vec{x},t)  = \sum_{k} v_{ka}(\vec{x};t) \tau_{k\alpha}^{[n,d]}(t)  $$

from the eigenvectors and perambulators.

### Parameters

| Parameter          | Type                       | Description                            |
|--------------------|----------------------------|----------------------------------------|
| `eigenvectors`     | `eigenPack`                | $v_k$                                  |
| `perambulator`     | `Perambulator<SpinVector>` | $\tau$                                 |
| `noise`            | `std::vector<Complex>`     | $\rho$                                 |
| `DistilParameters` | `std::vector&lt;int&gt;`   | TI,LI,SI,Nt,Nvec,Ns,nnoise,tsrc,Nt_inv |

### Potential changes

We should probably separate these modules, having one to create the source and one to create the sink vectors. We could additionally have a module to multiply one of these vectors by $\gamma_5$ to obtain $\bar{\varphi},\bar{\varrho}$

### Dependencies

- Laplacian eigenPack

- `perambulator` $\tau$

- `noises` $\rho$



### Products

- `source vectors` $\varrho$

- `sink vectors` $\varphi$

-----------


