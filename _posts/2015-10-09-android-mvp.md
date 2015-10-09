---
layout: post
title: Example of Model View Presenter pattern on Android
tags: android, mvp
comments: true
---
There're many interesting descriptions about the *Model View Presenter* (MVC) pattern and its ins and outs, for example [this](http://martinfowler.com/eaaDev/uiArchs.html) (at the end of the article).

It's a recommended pattern to **make views independent** and take most of logic out from your activities.

This post shows an extremely brief example about implementing MVP on Android.

<!--break-->

![MVP flow]({{site.url}}/img/mvp.png)

# Initial settings

The goal is simple, to update a progress indicator. To achieve it, you will use a couple of external libraries:

+ [CircleProgress](https://github.com/lzyzsd/CircleProgress) to draw the indicator
+ [Butter Knife](http://jakewharton.github.io/butterknife/) a view injection library that saves you a lot of boilerplate code

So, the build gradle file:

{% highlight javascript %}
apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.0"

    defaultConfig {
        applicationId "com.example.danilao.rxtest"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    lintOptions {
        disable 'InvalidPackage'
        abortOnError false
    }

    packagingOptions {
        exclude 'META-INF/services/javax.annotation.processing.Processor'
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.0.0'

    compile 'com.jakewharton:butterknife:7.0.1'
    compile 'com.github.lzyzsd:circleprogress:1.1.0@aar'
}
{% endhighlight %}

# The view

The unique interaction will be through a button. When you click it, the progress indicator will be updated:

{% highlight java %}
public interface MainView {
    void updateProgress(int progress);
}
{% endhighlight %}

# The presenter

The presenter has to complete the complex operations related to *guess* the new progress value and communicate the result to the view:

{% highlight java %}
public interface MainPresenter {
    void calculateProgress(int currentProgress);
}
{% endhighlight %}

It uses the current value to calculate the next one. However, in this example you'll work with a silly implementation:

{% highlight java %}
public class MainPresenterImpl implements MainPresenter {

    private MainView mainView;

    public MainPresenterImpl(MainView mainView) {
        this.mainView = mainView;
    }

    @Override
    public void calculateProgress(int currentProgress) {
        int finalValue = new Random().nextInt(100);
        mainView.updateProgress(finalValue);
    }
}
{% endhighlight %}

> For simplicity, in this example we're not defining a model.

# Final step

Finally, all connections are made into the `Activity` class:

{% highlight java %}
public class MainActivity extends AppCompatActivity implements MainView {

    @Bind(R.id.donut_progress) DonutProgress donut;
    @Bind(R.id.button) Button button;

    MainPresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);

        updateProgress(0);

        presenter = new MainPresenterImpl(this);
    }

    @OnClick(R.id.button)
    public void start() {
        presenter.calculateProgress(donut.getProgress());
    }

    @Override
    public void updateProgress(int newProgress) {
        donut.setProgress(newProgress);
    }
}
{% endhighlight %}

You get a clean view where the only logic is about layout binding and capturing user interactions (button taps). All the heavy operations are located inside the presenter.

In the my next post, I'll add some extra actions and I'll upload the entire project code.