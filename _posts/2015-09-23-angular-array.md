---
layout: post
title: AngularJS. Filling an array dynamically
tags: angular, angularjs, array, focus
comments: true
---

This little example based on **AngularJS** shows how to fill an array by creating input fields linked with each of its positions. Also, you'll see a fine alternative to avoid losing focus issues.

You can find a working snippet [here](https://jsfiddle.net/danilao/g3gw079n/18/).

<!--break-->

The code inside the **controller** is quite simple. It defines an `array` var to store the values and a couple of functions: one to add a new value and another to remove a certain position from the array:

{% highlight javascript %}        
$scope.array = [''];

$scope.addRow = function() {
    console.log('Add');
    $scope.array.push('');  
};
        
$scope.removeRow = function(position) {
    $scope.array.splice(position, 1);  
};
{% endhighlight %}

On the other hand, the layout looks like:

{% highlight html %}
<p>
    <button ng-click="addRow()">Add field</button>
</p>

<table>
    <tr>
        <th>Field</th>
        <th></th>
    <tr>
    <tr ng-repeat="field in array track by $index">
        <td><input type="text" ng-model="array[$index]"></td>       
        <td><button ng-click="removeRow($index)" ng-show="$index!=0">Remove</button></td>
    </tr>   
</table>
<p>
    <b>Current values</b>
    <br>
    <span>{{array}}</span>
</p>
{% endhighlight %}

The key is the line:

{% highlight html %}
<tr ng-repeat="field in array track by $index">
{% endhighlight %}

And specially `track by $index`, since a side effect of omitting it is every time you type a character **the input field loses the focus**. Also, another error you could find is:

```
Error: [ngRepeat:dupes] Duplicates in a repeater are not allowed. Use 'track by' expression to specify unique keys.
```

Check it [here](https://jsfiddle.net/danilao/zmvejy3y/).

For further info, take a look at the [official documentation](https://docs.angularjs.org/api/ng/directive/ngRepeat).