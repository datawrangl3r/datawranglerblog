---
title: 'Productionize ML Models - MNIST Handwritten Digits Prediction'
date: 2017-07-31T10:55:00.002-07:00
draft: false
aliases: [ "/2017/07/model-productionisation-mnist.html" ]

# post thumb
image: "images/featured-post/post1.jpg"

# meta description
description: "Demonstration of typical MNIST Handwritten Digits Prediction using Python and to productionize using Flask Microframework"

categories:
  - "Machine Learning"
tags:
  - "Machine Learning"
  - "Flask"
  - "Python"
  - "Training"
  - "Prediction"

# post type
type: "featured"
---

## Introduction

Building ML models have become child's play in recent days. With the advent of machine learning frameworks such as `tensorflow`, `pytorch` and `scikit-learn`, the ability to build a machine learning model has been brought down from days to minutes. In this article, we will be demonstrating how to productionize ML models using Python-based micro web framework `Flask`.

## Demonstration

This post isn't yet another post about MNIST Handwritten Digits Prediction. There are about hundreds of tutorials available on-line to tutor. <a href="https://www.oreilly.com/learning/not-another-mnist-tutorial-with-tensorflow" rel="nofollow noopener" target="_blank">Here's</a> a quick introduction, to understand all the mechanics of the prediction process in tensorflow for the MNIST datasets to help you get up and running.  
  
Now that we are done with the basics, it is time to head over to  <a href="https://github.com/datawrangl3r/mnistProduction" rel="nofollow noopener" target="_blank">https://github.com/datawrangl3r/mnistProduction</a> and clone the project.


  
We are about to deploy an image prediction RESTful API, powered by <a href="http://flask.pocoo.org/" rel="nofollow noopener" target="_blank">Flask</a> microframework.

The code in the repo was written and tested on Python V2.7.

```bash
## Step 1:
> python 1_modelCreate.py

## Step 2
> python 2_predict.py
```

Step 1 mentioned above, creates the model to identify the handwritten digits using `Tensorflow`. The output of the model is `mnist_model.ckpt` which is used by the second script `2_predict.py`. The devised model can be used to test the pictures of input digits found in the same repository. In the code, the `/predictint` route, is attached to the function which predicts the number of the image. The `2_predict.py` script is an API by itself served on port 5000. The `GET` requests take an argument `imageName` corresponding to the name of the image.

The project directory contains the `numerals_sample.jpg`, from which one may crop the required digits out. For this demo, we shall look at `numba3.jpg`, `numba5.jpg`, `numba6.jpg`, `numba7.jpg` and `numba9.jpg` present in the same directory.

With the `2_predict.py` script running, fire up the browser, and hit the following URL to test our model with the file `numba6.jpg`.

<a href="http://localhost:5000/predictint?imageName=numba6.jpg" rel="nofollow noopener" target="_blank">http://localhost:5000/predictint?imageName=numba6.jpg</a>
  
![Fig1: Number 6 in the testing](../../images/post/2-model-productionisation-mnist/img1.jpg)

The model has a pretty good accuracy rate. The response is given as number 6.  
  
![Fig2: Mind == blown](../../images/post/2-model-productionisation-mnist/img2.gif)

Let's evaluate the model with the image - `numba7.jpg`.  

![Fig3: Number 7 in the testing](../../images/post/2-model-productionisation-mnist/img3.jpg)

<a href="http://localhost:5000/predictint?imageName=numba7.jpg" rel="nofollow noopener" target="_blank">http://localhost:5000/predictint?imageName=numba7.jpg</a>

You may have noticed the response to be as 7, correctly predicted by the model as a response.

![Fig4: All Righty Then](../../images/post/2-model-productionisation-mnist/img4.gif)
 
Let's try out the image: `numba9.jpg`.

![Fig5: Number 9 in the testing](../../images/post/2-model-productionisation-mnist/img5.jpg)

[http://localhost:5000/predictint?imageName=numba9.jpg](http://localhost:5000/predictint?imageName=numba9.jpg)  

The response is received as number 5, which is weird for input as 9.

![Fig6: Uh oh.!](../../images/post/2-model-productionisation-mnist/img6.gif)

The prediction is wrong. However, Five does look a lot like 9 to the human eyes. 

![Fig7: Believe your eyes!](../../images/post/2-model-productionisation-mnist/img7.gif)

The model can be further trained and improved by adding more datasets to train on. Thus, our API can be really helpful in deploying the created model to production.

## Conclusion

In this article, we have learned how to deploy the devised model to production by creating an API that loads the model and predicts based on the input. The accuracy of the model can be improved over time by training with more datasets. A separate route can be written to facilitate this.