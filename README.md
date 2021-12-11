# ETAS
Non-Stationary/Non-homogeneous Epidemic Type Aftershock Sequence (ETAS) model

Code for ETAS models
1) Temporal ETAS
2) Spatial ETAS 
3) Spatio-temporal ETAS

The background rate is considered non-homogeneous in the respective domains. It is parameterised as a spline function.

Both Freqeuntist and Bayesian solutions are presented. 

For detailed description of the models see my [Ph.D. thesis](https://ngri.res.in/upload/thesis/Sasi%20Kumar%20Reddy_Thesis-2019.pdf).

Changes made after publication of thesis:

1) Slight change in the parameterization of the background rate. Instead of representing it as a spline f(x), it is represented as exponential of spline function exp(f(x)). This change helps in preventing underfitting large background rate values.

2) The above change necessitates a newer dimensionality reduction technique. QuadTree/OcTree based partitioning of the domain is used to achieve this. The background rate is expressed as a piece-wise constant function over such a partition.

3) A particularly expensive step in the fitting/training algorithm involves taking the inverse of a huge matrix. Suppose the background rate in 3D is expressed using 20 (Lat) x 20 (Long) x 100 (Time) splines, the said step requires inversion of 40000 x40000 matrix. Two such inversions are needed. Fortunately if we decide to just use the gradients (no hessians) to optimize the marginal log-likelihood (to estimate the optimal hyperparameters), we don't need the full inverse. we only need the inverse at the indices of the matrix where the elements are non-zero. This subset inversion can be performed using Dr. Tim Davis's [sparseinv](https://github.com/DrTimothyAldenDavis/SuiteSparse/tree/257b2ba2a5b40b893bff203626331b4b9c96d681/MATLAB_Tools/sparseinv) or [MUMPS](http://mumps.enseeiht.fr/doc/userguide_5.4.1.pdf#subsection.3.16). The latter is much faster but difficult to install and get it working with MATLAB.

4) In the thesis the parameters of the model and the hyperparameter are estimated using an iterative algorithm. That is, initial values of hyperparameters are assumed and parameters are estimated using MLE, these parameters are now held fixed and hyperparameters are estimated by optimizing marginal log-likelihood. These two steps are repeated one after the other until convergence. This is the standard procedure used in the literature (see for e.g., Ogata 2011, Kumazawa and Ogata 2014 etc). However, the inherent assumption here is that the hessian of the  maximum likelihood objective changes very little on changing the hyperparameters. Although such assumption is invalid, it greatly simplifies the problem to be solved. However, this makes the whole inversion/training/model fitting procedure unstable (Hessian matrices becoming non psd during optimization, different starting values lead to different optimal values etc). 

The proper approach would be to optimize the marginal log-likelihood while optimizing log-likelihood at each step. In other words, a nested iteration with an outer step of optimizing the marginal log-likelihood to estimate the hyperparameters and an inner step of maximum likelihood estimation to get optimal model parameters. However this requires computation of gradients and hessian of the solution of the inner optimization objective w.r.t to the hyperparameters. These can be computed following a procedure similar to the one in [Wood 2011](https://people.bath.ac.uk/man54/SAMBa/ITTs/ITT2/EDF/REMLWood2009.pdf).

Code for both the iterative method and the nested method are provided. Note that although nested method is better, the aforementioned subset inversion technique is not useful here. So if the background rate is expressed in terms of a large number of parameters, iterative estimation method (with sparse subset inversion enabled and hessians disables) is the only feasible option. In such cases, care must be taken to provide good initial parameter/hyperparameter values to the algorithm.

The STAN/C++ based MCMC algorithm to perform Bayesian estimation for these models is also provided. However, note that the code for Bayesian methods is not updated; it is still based on the previous parameterization of the background rate function. A few minor changes are needed to make it compatible with the newer formulations.

[TODO] Clean the code!
