---
layout: post
title:  "Empty"
date:   2021-06-20 14:40:16 +1200
categories: nothing
---

Empty blog post
Hey cool it's a code snippet

{% highlight rust %}
// Reads a buffer into a String Vec.
fn read_bytes(bytes: &[u8]) -> Vec<String> {
    return String::from_utf8_lossy(bytes)
        .split("\n")
        .map(|val| val.to_owned())
        .collect();
}


{% endhighlight %}
