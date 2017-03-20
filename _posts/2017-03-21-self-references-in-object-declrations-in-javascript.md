---
layout: post
title: Self-referencing keys in a JavaScript object declarations 
tags: javascript, programming languages
---

Often times while programming in JavaScript, which I have been doing a lot lately, I find myself wanting to refer to a key that I have just declared when I am initializing an object.

{% highlight javascript %}
const myNewObj = {
  a: 'xxx',
  b: 'yyy',
  c: a + this.b, // doesn't work
};

console.log(myNewObj.c); // undefined
{% endhighlight %}

Here, neither `a` or `this.b` would work since there is no `a` yet while `c` is being defined and `this` does not refer to `myNewObj`. Obviously, you could define `xxx` and `yyy` in separate variables first and refer to them later in the object initialization. Or you could initialize the object without `c` key and later define `c` as `myNewObj.c = myNewObj.a + myNewObj.b;`. But a neater way that I found out is to utilize `getters`.


{% highlight javascript %}
const myNewObj = {
  a: 'xxx',
  b: 'yyy',
  get c() {
    return this.a + this.b;
  }
};

console.log(myNewObj.c); // 'xxxyyy'
{% endhighlight %}
Pretty neat, isn't it!

