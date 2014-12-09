---           
layout: post
title: Simple Linear Regression With Incanter and Clojure
date: 2014-09-09 21:15:17 UTC
updated: 2014-09-09 21:15:17 UTC
comments: false
categories: clojure incanter machine-learning
---

Linear regression common statistical method that can be applied when we need to express the mathematical relationship between two variables.

If there seems to be a linear relation between two variables and this relation seems deterministic, linear regression could be a simple and efficient choice, at least for the beginning. 

In this example, we have a training set that contains housing prices and we want to come up with a linear equation that fits this data set as closely as possible. 

In the scope of this post we will have a single dimension. We will simply assume that the price of the house is purely based on the lotsize. We will take other features in consideration in the future posts.

We have this file "housing.csv" taken from [here](http://vincentarelbundock.github.io/Rdatasets/datasets.html) and it looks something like the following:

		"no",price,lotsize,bedrooms,bathrms,stories,driveway,recroom,fullbase,gashw,airco,garagepl,prefarea
		1,42000,5850,3,1,2,yes,no,yes,no,no,1,no
		2,38500,4000,2,1,1,yes,no,no,no,no,0,no
		3,49500,3060,3,1,1,yes,no,no,no,no,0,no
		4,60500,6650,3,1,2,yes,yes,no,no,no,0,no

We will create two vectors after parsing this file, X and Y. The vector Y will contain the price information and X will contain the lotsize:

		(def X (atom []))
		(def Y (atom []))

		(defn parse-single-tsv-line
		  [^String line]
		  (let [content (string/split line #"\,")
		        content-count (count content)]
		    (if (and (= 13 content-count) (not (.startsWith line "\"")))
		      (let [pri (read-string (nth content 1))
		            size (read-string (nth content 2))]
		        {:price pri :lotsize size})
		      nil)))

		(defn parse-tsv-and-create-matrices!
		  [tsv-path]
		  (with-open [rdr (clojure.java.io/reader tsv-path)]
		    (let [lines (line-seq rdr)]
		      (dorun
		        (pmap #(let [parsed (parse-single-tsv-line %1)]
		                (if-not (nil? parsed)
		                  (do
		                    (swap! X conj (:lotsize parsed))
		                    (swap! Y conj (:price parsed)))))
		              lines)))))

		(parse-tsv-and-create-matrices! "test-resources/ml/housing.csv")

Now once we have these vectors in place ( Mx1 matrices) we can plot them using incanter easily like the following:

		(defn make-scatter-plot-chart [X Y]
		  (charts/scatter-plot X Y))

		(defn display [X Y]
		  (inct/view (make-scatter-plot-chart X Y)))

This should display an image something like the following:
![Plot1](/assets/plot1.png)

Looks nice but so far we have just displayed the original data. We should come up with a model that fits this dataset. You can use the linear-model function in the incanter.stats namespace to perform the regression:

		(defn ols-linear-model [Y X]
  			(st/linear-model Y X))

  		(ols-linear-model @Y @X)

Executing this function on our single-dimension matrices will return a clojure map containing the result of the regression. With my dataset, the coefficients i get back looks like the following:

		(:coefs (ols-linear-model @Y @X) )
		=> [34077.20668646169 6.610220373141942]

We are done already, but just in case you want to plot the result, the map will contain an entry indentified by :fitted keyword, which are the predicted values of Y. And you can ploy it using a function like the one below

		(defn plot-model [X Y] (inct/view
                         (charts/add-lines (make-scatter-plot-chart X Y)
                                           X (:fitted (ols-linear-model Y X)))))
		(plot-model @X @Y)

which creates a graph that looks something like the following:

![Plot2](/assets/plot2.png)

Have fun playing with incanter in the repl!

