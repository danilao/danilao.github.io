---
layout: post
title: Animations with RxAndroid
tags: android, mvp, rx, animations
comments: true
---

In the [previous post]({{site.url}}/post/android-mvp) you saw a brief example about *Model View Presenter* pattern and how to apply it to update the value of a progress indicator.

In this post you'll go further and will animate the transition between the initial and final progress values and will include a couple o new functions to your view.

<!--break-->

In order to achieve the animation you'll use a functional-reactive approach through [RxAndroid](https://github.com/ReactiveX/RxAndroid) and [Retrolambda](https://github.com/orfjackal/retrolambda), so let's add them to your project.

Include retrolambda at the top of your gradle file as a plugin:

{% highlight javascript %}
buildscript {
    repositories {
        mavenCentral()
    }

    dependencies {
        classpath 'me.tatarka:gradle-retrolambda:2.5.0'
    }
}

apply plugin: 'me.tatarka.retrolambda'
{% endhighlight %}

Now is the turn of Retrolambda. Add the next lines into your `dependencies` section:

{% highlight javascript %}
compile 'io.reactivex:rxandroid:1.0.1'
compile 'io.reactivex:rxjava:1.0.14'
{% endhighlight %}

# Going reactive

Your objective is create a *frame by frame* animation between the initial and final progress values. You already know them, so your main task is calculate the middle points and update the indicator each certain time to get the animation effect.

{% highlight java %}
int finalValue = new Random().nextInt(100);
int difference = finalValue - currentProgress;
int direction = difference > 0 ? 1 : -1;

Integer numbers[] = new Integer[Math.abs(difference)];

for(int i = 0; i < Math.abs(difference); i++) {
    numbers[i] = currentProgress + direction * i;
}
{% endhighlight %}

First, the difference between the values is calculated and then, you create and array, `numbers`, that contains the middle values.

Now, define two **Observables**, one that emits for each value in the `numbers` array:

{% highlight java %}
Observable<Integer> values = Observable.from(numbers);
{% endhighlight %}

And another one that emits values every 30 milliseconds:

{% highlight java %}
Observable<Long> interval = Observable.interval(30, TimeUnit.MILLISECONDS);
{% endhighlight %}

The goal is to combine them into an unique observable that emits the correct middle values each X time to achieve the animation effect.

Thanks to RxJava, it's possible through the [Zip operator](http://reactivex.io/documentation/operators/zip.html):

![Zip operator]({{site.url}}/img/zip.png)

So the code:

{% highlight java %}
Observable.zip(values, interval, (animationValue, aLong) -> animationValue)
        .subscribeOn(Schedulers.io())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(animationValue -> mainView.updateProgress((int)animationValue));
{% endhighlight %}

Let's see operator by operator:

+ `zip()` combines the two observables and emits the middle values each 30 milliseconds.
+ `subscribeOn()` indicates that the operations have to be done in a separete thread.
+ `observeOn()` to receive the result in the main thread. This is useful in order to update the user interface.
+ `subscribe()` indicates the action to be triggered when a value is observed. In this case, you only update the progress view.

Now if you run the project, you'll see the animated transition between values every time the button is tapped. However, the button is never disabled, you're able to tap it multiple times and it produces ugly results, so let's fix it.

# Final touches

You need a mechanisms to disable the button while the animation is running. The first step is include the new functions in the View definition:

{% highlight java %}
public interface MainView {
    void updateProgress(int progress);
    void disableButton();
    void enableButton();
}
{% endhighlight %}

The implementation is quite straightfordward

{% highlight java %}
 @Override
    public void enableButton() {
        button.setEnabled(true);
    }

    @Override
    public void disableButton() {
        button.setEnabled(false);
    }
{% endhighlight%}

At this time, the las step is to include them in the correct place within the presenter:

{% highlight java %}
@Override
public void calculateProgress(int currentProgress) {
    mainView.disableButton();

    // rest of the implementation

    Observable.zip(values, interval, (animationValue, aLong) -> animationValue)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .doOnCompleted(() -> mainView.enableButton())
            .subscribe(animationValue -> mainView.updateProgress((int)animationValue));
{% endhighlight %}

Just after tapping the button, it will be disabled. When all the process is over, the operator `doOnCompleted()` enables the button again.

You can find all the code [here](https://github.com/danilao/android-mvp-rx-animation-example).
