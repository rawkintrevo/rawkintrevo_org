---
id: 1672
title: 'Introducing Pre-canned Algorithms in Apache Mahout'
date: '2017-05-02T13:46:03+00:00'
author: rawkintrevo
layout: post
guid: 'http://rawkintrevo.org/?p=1672'
permalink: /2017/05/02/introducing-pre-canned-algorithms-apache-mahout/
jabber_published:
    - '1493735469'
email_notification:
    - '1493735471'
publicize_twitter_user:
    - rawkintrevo
publicize_google_plus_url:
    - 'https://plus.google.com/107169808969912762220/posts/b2a87YvQCnV'
publicize_linkedin_url:
    - 'https://www.linkedin.com/updates?discuss=&scope=94269141&stype=M&topic=6265180717278384128&type=U&a=p4HU'
image: /wp-content/uploads/2017/05/img_20170502_092805.jpg
categories:
    - 'big data for noobs'
    - Engineering
    - mahout
---

Apache Mahout v0.13.0 is out and there are a lot of exciting new features and integration including GPU acceleration, Spark 2.x/Scala 2.10 integration (experimental- full blown in 0.13.1), and a new framework for “precanned algorithms”. In this post we’re going to talk about the new algorithm framework, and how you can contribute to your favorite machine learning on Big Data library.

If you’re not familiar with the Apache Mahout project, it might be helpful to [watch this video](https://rawkintrevo.org/2016/12/19/apache-mahout-whats-next-dfw-big-data-meetup-talk/) but in short- it allows you to quickly and easily write your own algorithms in a distributed back-end independent (think Apache Spark, Apache Flink, etc), and mathematically expressive extension of the Scala language. Now v0.13.0 allows the user to accelerate their distribute cluster with GPUs (this is independent of Spark- ANY cluster can be accelerated), and lays out a framework of pre-canned algorithms.

# Key Concepts

The Algorithms Framework in Apache Mahout, borrows from the traditions of many of the great machine learning and statistical frameworks available today, but most notably- R and Python’s sklearn. When reasonable, Mahout makes a good faith effort to draw on the best parts of each of these.

- *sklearn –* has a very consistent API.
- ***R*** *–* is very flexible.
- Both are extendable, and encourage users to create and submit their own implementations to be available for other users (via CRAN and Pypi respectively).

### Fitters versus Models

The first concept we want to address is the idea of a fitter and a model. Now that I have setup the Mahout Algorithms framework, I instantly point out a major break from the way things are done in R and sklearn. As the great thinkers Ralph Waldo Emerson and the person who wrote [PEP-8](https://www.python.org/dev/peps/pep-0008/) said, “A foolish consistency is the hobgoblin of little minds.”

In sklearn, the model and the fitter are contained in the same class. In R, there is an implicitly similar paradigm… sometimes.

Model is an object which contains the parameter estimates. The `R` function `lm` generates models. In this way, a `Fitter` in Apache Mahout generates a model of the same name (by convention. E.g. `OrdinaryLeastSquares` generates an `OrdinaryLeastSquaresModel` which contains the parameter estimates and a `.predict(drmX)` method for predicting new values based on the model.

Recap: ***A Fitter Generates a Model. A model is an object that contains the parameter estimates, fit statistics, summary, and a `predict()` method.***

Now if you’re thinking, “but why?”, good on you for questioning things. Why this break from sklearn? Why not let the fitter and the model live in the same object? The answer is because at the end of the day- we are dealing in big data, and we want our models to be serialized as small as is reasonable. If we were to include everything in the same object (the fitter, with the parameter estimates, etc.) then when we saved the model or shipped it over the network we would have to serialize all of the code required to fit the model and ship that with it. This would be somewhat wasteful.

#### Class Heirarchy

The following will make the most sense if you understand class hierarchy and class inheritance. If you don’t know/remember these things, now would be a good time to review.

![Screen Shot 2017-03-30 at 10.27.34 AM](https://i0.wp.com/rawkintrevo.org/wp-content/uploads/2017/05/screen-shot-2017-03-30-at-10-27-34-am.png?resize=685%2C459&ssl=1)

This isn’t a complete diagram but it is illustrative. For example- all `Model`s have a `summary` string. All `SupervisedModels` have a `Map` of `testResults`. All `RegressorModel`s have a `.predict(...)` method, and when `ClassifierModel` is introduced, they may have a `.predict(...)` method as well, or perhaps they will have a `.classify(...)` method.

*Preprocessors* are treated as unsupervised models. They must also be fit. Consider a `StandardScaler`, which must be “fit” on a data set to learn the mean and standard deviation.

The hierarchy of fitters is identical.

#### Hyper-parameter system

Hyper-parameters are passed in fitter functions as symbols. For example:

\[code lang=scala\]  
val model = new OrdinaryLeastSquares\[Int\]().fit(drmX, drmY, 'calcCommonStatistics → false)  
\[/code\]

Different methods have different hyper-parameters which maybe set. This method has advantages of extreme flexibility. It also side-steps the type safety of the Scala language, which depending on weather or not you like or hate type-safety, you might consider to be a good or bad thing. A notable draw back- if you pass a parameter that isn’t used by the method, it will be ignored ***silently***, that is to say it will be ignored and it won’t warn you are throw an error. The real threat here is typos- where you think are doing something like, specifying an interceptless regression, however instead of specifying `'addIntercept -&gt; false` you accidentally type `'addInterept -&gt; false`, then the regression will add an intercept and throw no warnings that you’ve committed a typo. (This will possibly be fixed soon).

Also, in both hyperparameter examples given have had `Boolean` values, however the value can be anything. For example, in Cochrane-Orcutt on of the hyperparameters `'regressor` can be any sub-class of `LinearRegressorFitter`!

# In Practice

##### Preprocessors

There are currently three *pre-processors* available.  
\* `AsFactor` which is sometimes referred to as Dummy Variables or One-Hot encoder (Mahout chose R-semantics here over Python)  
\* `StandardScaler` which is goes by the same name in sklearn and the function `scale` in R.  
\* `MeanCenter` which is very similar to the standard scaler, however it only centers each column. In the future it is possible that MeanCenter will be combined with StandardScaler (as is done in R).

##### A preprocessor example

A fun tip: the unit tests of any package are full of great example. This one comes from: <https://github.com/apache/mahout/blob/master/math-scala/src/test/scala/org/apache/mahout/math/algorithms/PreprocessorSuiteBase.scala>

###### Setup Code

\[code lang=scala\]  
val A = drmParallelize(dense(  
(3, 2, 1, 2),  
(0, 0, 0, 0),  
(1, 1, 1, 1)), numPartitions = 2)

// 0 -&gt; 2, 3 -&gt; 5, 6 -&gt; 9  
\[/code\]

###### How to use `AsFactor` from Apache Mahout

\[code lang=scala\]  
val factorizer: AsFactorModel = new AsFactor().fit(A)  
val factoredA = factorizer.transform(A)  
val myAnswer = factoredA.collect  
\[/code\]

###### Check our results

\[code lang=scala\]  
println(factoredA)  
println(factorizer.factorMap)  
val correctAnswer = sparse(  
svec((3 → 1.0) :: (6 → 1.0) :: (8 → 1.0) :: (11 → 1.0) :: Nil, cardinality = 12),  
svec((0 → 1.0) :: (4 → 1.0) :: (7 → 1.0) :: ( 9 → 1.0) :: Nil, cardinality = 12),  
svec((1 → 1.0) :: (5 → 1.0) :: (8 → 1.0) :: (10 → 1.0) :: Nil, cardinality = 12)  
)  
val epsilon = 1E-6  
(myAnswer.norm – correctAnswer.norm) should be &lt;= epsilon  
(myAnswer.norm – correctAnswer.norm) should be &lt;= epsilon  
\[/code\]

The big call out from the above- is that the interface for this preprocessor (the second block of code) is exceptionally clean for a distributed, GPU accelerated, machine learning package.

##### Regressors

There are currently two regressors available:  
\* `OrdinaryLeastSquares` – Closed form linear regression  
\* `Cochrane-Orcutt` – A method for dealing with Serial Correlation

Oh, horay- *another* linear regressor for big data. First off- don’t be sassy. Second, OLS in Apache Mahout is closed form- that is to say, it doesn’t rely on Stochastic Gradient Descent to approximate the parameter space ***β***.

Among other things, this means we are able to know the standard errors of our estimates and make a number of statistical inferences, such as the significance of various parameters.

For the initial release of the algorithms framework, `OrdinaryLeastSquares` was chosen because of its widespread familiarity. `CochraneOrcutt` was chosen for its relative obscurity (in the Big Data Space). The [Cochrane Orcutt procedure](https://en.wikipedia.org/wiki/Cochrane%E2%80%93Orcutt_estimation) is used frequently in econometrics to correct for auto correlation in the error terms. When auto-correlation (sometimes called serial-correlation) is present the standard errors are biased, and so is our statistical inference. The Cochrane Orcutt procedure attempts to correct for this.

It should be noted, implementations of Cochrane-Orcutt in many statistics packages such as R’s `orcutt` iterate this procedure to convergence. This is ill-advised on small data and big data alike. [Kunter et. al](https://www.amazon.com/Applied-Linear-Statistical-Models-Hardcover/dp/B010EWX85C/) recommend no more than three iterations of the Cochrane Orcutt procedure- if suitable parameters are not achieved, the user is advised to use another method to estimate *ρ*.  
``

The point of implementing the `CochraneOrcutt` procedure was to show, that the framework is easily extendable to esoteric statistical/machine-learning methods, and users are encouraged to extend and contribute. [Observe the implementation](https://github.com/apache/mahout/blob/master/math-scala/src/main/scala/org/apache/mahout/math/algorithms/regression/CochraneOrcuttModel.scala#L94) of the algorithm, and after groking, the reader will see that the code is quite expressive and tractable, and the majority of the `fit` method is dedicated to copying variables of interest into the resulting `Model` object.

##### A Regression Example

###### Setup Code

\[code lang=scala\]  
val alsmBlaisdellCo = drmParallelize( dense(  
(20.96, 127.3),  
(21.40, 130.0),  
(21.96, 132.7),  
(21.52, 129.4),  
(22.39, 135.0),  
(22.76, 137.1),  
(23.48, 141.2),  
(23.66, 142.8),  
(24.10, 145.5),  
(24.01, 145.3),  
(24.54, 148.3),  
(24.30, 146.4),  
(25.00, 150.2),  
(25.64, 153.1),  
(26.36, 157.3),  
(26.98, 160.7),  
(27.52, 164.2),  
(27.78, 165.6),  
(28.24, 168.7),  
(28.78, 171.7) ))

val drmY = alsmBlaisdellCo(::, 0 until 1)  
val drmX = alsmBlaisdellCo(::, 1 until 2)  
\[/code\]

###### Cochrane-Orcutt

\[code lang=scala\]  
var coModel = new CochraneOrcutt\[Int\]().fit(drmX, drmY , ('iterations -&gt; 2))  
println(coModel.beta)  
println(coModel.se)  
println(coModel.rho)  
\[/code\]

##### Regression Tests

Unlike R and sklearn, all regression statistics should be considered optional, and very few are enabled by default. The rationale for this is that when working on big data, calculating common statistics could be costly enough that, unless the user explicitly wants this information, the calculation should be avoided.

The currently available regression tests are  
\* `CoefficientOfDetermination` – calculated by default, also known as the R-Square  
\* `MeanSquareError` – calculated by default, aka MSE  
\* `DurbinWatson` – not calculated by default, a test for the presence of serial correlation.

When a test is run, the convention is the following:

\[code lang=scala\]  
var model = …  
model = new MyTest(model)  
model.testResults.get(myTest)  
\[/code\]

The model is then updated with the test result appended to the model’s `summary` string, and the value of the test result added to the model’s `testResults` `Map`.

# Extending

Apache Mahout’s algorithm framework was designed to be extended. Even with the few example given, it should be evident that it is much more extensible than SparkML/MLLib and even sklearn (as all of the native optimization is abstracted away).

While the user may create their own algorithms with great ease- all are strongly encouraged to contribute back to the project. When creating a “contribution grade” implemenation of an algorithm a few considerations must be taken.

1. The algorithm must be expressed purely in Samasara (The Mahout R-Like DSL). That is to say, the algorithm may not utilize any calls specific to an underlying engine such as Apache Spark.
2. The algorithm must fit into the existing framework or extend the framework as necessary to ‘fit’. For example, we’d love to see a classification algorithm, but one would have to write the `Classifier` trait (similar to the `Regressor` trait).
3. New algorithms must demonstrate a prototype in either R, sklearn, or someother package. That isn’t to say the algorithm must exist (though currently, all algorithms have an analgous R implementation). If there is no function that performs your algorithm, you must create a simple version in another language and include it in the comments of your unit test. This ensures that others can easily see and understand what it is that the algorithm is supposed to do.

Examples of number three are abound in the current unit tests. [Example](https://github.com/apache/mahout/blob/master/math-scala/src/test/scala/org/apache/mahout/math/algorithms/RegressionTestsSuiteBase.scala#L90)

# Conclusions

Apache Mahout v0.13.0 offers a number of exciting new features, but the algorithms framework is (biasedly) one of my favorite. It is an entire framework that encourages statisticians and data scientists who have until now been intimidated by contributing to open source a green field opportunity to implement their favorite algorithms and commit them to a top-level Apache Software Foundation project.

There has been to date a mutually exclusive choice between ‘powerful, robust, and extendable modeling’ and ‘big data modeling’, each having advantages and disadvantages. It is my sincere hope and believe that the Apache Mahout project will represent the end of that mutual exclusivity.