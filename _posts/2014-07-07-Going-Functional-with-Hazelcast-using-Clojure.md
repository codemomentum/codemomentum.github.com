---           
layout: post
title: Going Functional with Hazelcast using Clojure
date: 2014-07-07 20:14:17 UTC
updated: 2014-07-07 20:14:17 UTC
comments: false
categories: clojure hazelcast distributed-programming
---


We have been using Hazelcast in my company for different projects quite a while now and i can say that it saves a lot of our time. This post is about an abstraction layer that is built on top of Hazelcast which tries to provide first class distributed operations and data structures for Clojure.


###Rationale
While being a ready to use data grid, Hazelcast can be used as a foundation for building data oriented distributed systems. 

There are lots of great fundamental building blocks in HazelCast, distributed maps, queues, query engine, map reduce etc.. I think that an application using these building blocks can be much more powerful in an ecosystem that is more data centric and simple than Java.

So, Clojure to the rescue? It has great abstractions, its data centric, Shortly, its the sane environment we have been looking for a while.

**clj-hazelcast** is a simple wrapper a around Hazelcast and provides some convenient stuff while working with clojure data structures. I implemented the map reduce support and there is an ongoing PR, you can find the latest source here in the meantime:

	https://github.com/codemomentum/clj-hazelcast

###Map Reduce Using Clojure Functions

clj-hazelcast defines some simple macros for distributed operations such as Map, Reduce, Combine. The macros are about capturing the defined function's string representation which is used later on to serialize/de-serialize arbitrary Clojure functions.

Here is a quick look at a mapper:

	(defmapper sample-mapper [k v] [[k (+ 1 v)])

This is a simple mapper that gets each key-value pair and returns a pair of k and value+1
Mapper is a function of two arguments: key and value, and must return a sequence of pairs. 

Similarly a reducer:

	(defreducer sample-reducer [k v acc] (if (nil? acc) 1 (inc acc)))
	
Reducer is a function of three arguments: key,value and an accumulator and it should return the new accumulator at the end.

So based on what we have already, here is how a simple wordcount looks in clj-hazelcast:

	(hz/put! @wordcount-map :k1 "clojure java")
    (hz/put! @wordcount-map :k2 "java clojure")
    (hz/put! @wordcount-map :k3 "lisp clojure")

	(mr/defmapper 
	mapper
    [k v]
     (let [words (re-seq #"\w+" v)]
      (partition 2 (interleave words (take (count words) (repeatedly (fn [] 1)))))))
	
	(mr/defreducer reducer [k v acc] (if (nil? acc) v (+ acc v)))
	
	=>
	{"clojure" 3 "java" 2 "lisp" 1}

You can find detailed examples in:
	
	clj-hazelcast.test.distributed-test
	clj-hazelcast.test.mr-test
	
###Distributed Queries Using Clojure Functions

Similar to the mappers and reducers,you can use defpredicate macro to use clojure functions as predicates.

	(require '[clj-hazelcast.query :as q])
	(q/defpredicate stark? [k v] (.contains v "Stark"))
	(q/values @query-map stark?)

###Summary

It seems Hazelcast is adding Aggregation module to bring down the level of complexity while writing MapReduce code, but luckily we already have a simple abstraction layer now through clojure which can be used to address similar problems.

This implementation is at an early stage so let me know if you have any ideas to make it better.

###Resources
Repo containing the latest MR support

	https://github.com/codemomentum/clj-hazelcast

Original Repo

	https://github.com/runa-labs/clj-hazelcast
 	
HZ Mapreduce Examples
	
	https://github.com/noctarius/hz-map-reduce/tree/master/src/main/java/com/hazelcast/example/mapreduce
	
	