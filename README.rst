-------------
MKLpy
-------------


MKLpy is a framework for Multiple Kernel Learning and kernel machines scikit-compliant.

This package contains:

* MKL algorithms
  * EasyMKL
  * RM-GD
  * R-MKL
  * Average of kernels

* a meta-MKL-classifier used in multiclass problems according to one-vs-one pattern;

* tools to operate over kernels, such as normalization, centering, summation, mean...;

* metrics, such as kernel_alignment, radius...;

* kernel functions, such as HPK and boolean kernels (disjunctive, conjunctive, DNF, CNF).



For more informations about classification, kernels and predictors visit `scikit-learn <http://scikit-learn.org/stable/>`_ docs.


Installation
-------------

You can easily install via this command:

.. code-block:: bash

 pip install MKLpy
 
If you do not have sudo privileges, try:

.. code-block:: bash

 pip install MKLpy --user
 

Requirements
------------

To work properly, MKLpy requires:

* numpy

* scikit-learn

* cvxopt


-------------
Examples
-------------

In the following sections, we show how the various tools in ``MKLpy`` could be used to generate, learn and evalute MKL alogrithms. Let's get started with loading a dataset scikit-learn, exploiting the svmlight standard

.. code-block:: python

    from sklearn.datasets import load_svmlight_file
    X,Y = load_svmlight_file(path)
    X = X.toarray()	#Important! MKLpy require dense matrices!

*Note* MKLpy requires dense matrices (not sparse) to operate!


Preprocessing
-------------

MKLpy provides several tools to preprocess data, some examples are:

.. code-block:: python

    from MKLpy.regularization import normalization,rescale_01
    X = rescale_01(X)
    X = normalization(X)

It is also possible to operate on kernels directly

.. code-block:: python

    from MKLpy.metrics.pairwise import HPK_kernel
    K = HPK_kernel(X,degree=2)

    from MKLpy.regularization import 	\
        kernel_centering,		\
        kernel_normalization,		\
        tracenorm
    Kc = kernel_centering(K)
    Kn = kernel_normalization(K)
    Kt = tracenorm(K)


Generation of Kernels
--------------------------

MKL algorithms requires a list or arrays of kernels. With tools from MKLpy, it is easy to create a list of kernels, or another custom list of your choice:

.. code-block:: python

    KL = [HPK_kernel(X,degree=d) for d in range(1,11)]
    
    #creating lists of boolean kernels
    from MKLpy.metrics.pairwise import			\
        monotone_conjunctive_kernel as mCK,		\
        monotone_disjunctive_kernel as mDK
    #WARNING: boolean kernels require binary valued data {0,1}
    KL = [mCK(X,k=d) for d in range(1,11)] + [mDK(X,k=d) for d in range(2,11)]


Learning of Kernels
--------------------------

The learning phase consists on two steps: 

 - learning kernels and 
 - fitting models by using a MKl algorithm and a standard kernel machine

.. code-block:: python

    from MKLpy.algorithms import EasyMKL,RMGD,RMKL,AverageMKL
    #learn kernels
    K_easy = EasyMKL(lam=0.1).arrange_kernel(KL,Y)
    K_rmgd = RMGD(max_iter=3).arrange_kernel(KL,Y)
    #fit models
    from sklearn.svm import SVC
    from MKLpy.algorithms import KOMD
    clf_komd = KOMD(lam=0.1,kernel='precomputed').fit(K_easy,Y)
    clf_svc  = SVC(C=10,kernel='precomputed').fit(K_rmgd,Y)

Now, we show a more simpler procedure, where MKL algorithms use a default base learner

.. code-block:: python

    clf = EasyMKL().fit(KL,Y)
    clf = AverageMKL().fit(KL,Y)

It is also possible to set a custom base learner

.. code-block:: python

    clf = EasyMKL(estimator=SVC(C=1)).fit(KL,Y)


Evaluation
-------------

You can evaluate a model by splitting a kernel list into train and test sets:

.. code-block:: python

    from MKLpy.model_selection import train_test_split, cross_val_score
    from sklearn.metrics import roc_auc_score
    
    KLtr,KLte,Ytr,Yte = train_test_split(KL,Y,train_size=.75,random_state=42)
    y_score = clf.fit(KLtr,Ytr).decision_function(KLte)
    auc_score = roc_auc_score(Yte, y_score)

Or using a cross-validation procedure: 

.. code-block:: python

    clf = EasyMKL(estimator=SVC())
    scores = cross_val_score(KL,Y,estimator=clf,n_folds=5)

Other useful tools
--------------------------

MKLpy contains a wider set of tools for kernel learning and MKL, of which some examples include computing the distance between classes as well as radius of an MEB:

.. code-block:: python

    from MKLpy.metrics import margin, radius
    K = AverageMKL().arrange_kernel(KL,Y)
    rho = margin(K,Y)	#distance between classes
    R = radius(K)	#radius of MEB
