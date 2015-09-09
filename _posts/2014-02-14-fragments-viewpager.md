---
layout: post
title: Android tutorial. How to replace fragments inside a ViewPager
tags: android, viewpager, fragment, tutorial
comments: true
---

> This is a migrated post from my former blog

I've developed a little example app about replacing a `Fragment` by another one inside a [ViewPager](http://developer.android.com/reference/android/support/v4/view/ViewPager.html). Also, if you navigate between them it's possible to keep the *backstack*:

<iframe width="700" height="315" src="https://www.youtube.com/embed/0n5F8VLqz2w" frameborder="0" allowfullscreen></iframe>

<!--break-->

&nbsp;

The key is to put the `ViewPager` inside a `Fragment` that acts as a container and that we'll use as a reference. Inside the `PageAdapter` class:

{% highlight java %}
// PagerAdapter class
public class SlidePagerAdapter extends FragmentPagerAdapter {
    public SlidePagerAdapter(FragmentManager fm) {
        super(fm);
    }
  
    @Override
    public Fragment getItem(int position) {
        /*
         * IMPORTANT: This is the point. We create a RootFragment 
         * acting as a container for other fragments
         */
        if (position == 0)
           return new RootFragment();
        else
           return new StaticFragment();
    }
  
    @Override
    public int getCount() {
        return NUM_ITEMS;
    }
}
{% endhighlight %}

In the `RootFragment` class:

{% highlight java %}
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container,
Bundle savedInstanceState) {
    /* Inflate the layout for this fragment */
    View view = inflater.inflate(R.layout.root_fragment, container, false);
      
    FragmentTransaction transaction = getFragmentManager().beginTransaction();
     
    /*
     * When this container fragment is created, we fill it with our first
     * "real" fragment
     */
    transaction.replace(R.id.root_frame, new FirstFragment());
      
    transaction.commit();
      
    return view;
}
{% endhighlight %}

And finally, inside the `FirstFragment` class, we can include links to another fragments:

{% highlight java %}
btn.setOnClickListener(new OnClickListener() { 
    @Override
    public void onClick(View v) {
        FragmentTransaction trans = getFragmentManager().beginTransaction();
        /*
         * IMPORTANT: We use the "root frame" defined in
         * "root_fragment.xml" as the reference to replace fragment
         */
        trans.replace(R.id.root_frame, new SecondFragment());
        
        /*
         * IMPORTANT: The following lines allow us to add the fragment
         * to the stack and return to it later, by pressing back
         */
        trans.setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
        trans.addToBackStack(null);
  
        trans.commit();
    }
});
{% endhighlight %}

The complete example is in [Github](https://github.com/danilao/fragments-viewpager-example) for complete free use and also there's an example app on [Google Play](https://play.google.com/store/apps/details?id=com.pineappslab.frcontainer).