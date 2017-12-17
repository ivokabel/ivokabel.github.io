---
title: Formatting Test
layout: post
---

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nunc ultricies sit amet nisl nec volutpat. Nam vel scelerisque ligula. Quisque et erat ac velit finibus ullamcorper ut scelerisque magna. Vivamus id est non metus feugiat viverra at vitae purus. Quisque nec aliquet mauris. Proin auctor vulputate risus. Proin a viverra augue, eget hendrerit nisl. Aenean vitae ex ac est dapibus posuere sed ac eros. Sed vitae ornare quam.

Nullam aliquam viverra mollis. Nam quis lacus diam. Suspendisse at nibh turpis. Aenean id vulputate nulla. Maecenas pulvinar ligula non vestibulum accumsan. Nam eget risus nec libero rhoncus convallis vel at nunc. In hac habitasse platea dictumst. In hac habitasse platea dictumst. Sed tincidunt dui non pulvinar auctor. Nullam et efficitur est. Donec mi tellus, dapibus eu tellus in, porta efficitur purus. Phasellus nec augue ac diam faucibus semper. Cras eleifend suscipit enim, ut posuere massa scelerisque id.

List:

* Item 1
* Item 2

Ordered list:

1. Item 1
   1. Sub-item 1
   2. Sub-item 2
2. Item 2

Code (C++):

```c++
if (isMatch){
  return true
}
```

Typography: - -- --- +− ...

## Math

Some math will be necessary as well. Since GitHub Pages partially disabled MathJax (for "security reasons"), I need to hack it in somehow and find a way which will work consistently both in my editor and on the resulting web:

### Inline

Single dollar: $\int_{S^2}f_s(\omega)\cos(\theta)d\omega$.

~~Round braces:~~ \\(\int_{S^2}f_s(\omega)\cos(\theta)d\omega\\).

Double dollar: $$\int_{S^2}f_s(\omega)\cos(\theta)d\omega$$

### Displayed

~~Square braces:~~ \\[\int_{S^2}f_s(\omega)\cos(\theta)d\omega\\]

~~Double dollar block on the next line:~~
$$
\int_{S^2}f_s(\omega)\cos(\theta)d\omega
$$

Double dollar block as a separate paragraph:

$$
\int_{S^2}f_s(\omega)\cos(\theta)d\omega
$$

Double dollar block in raw on the next line:
{% raw %}
$$
\int_{S^2}f_s(\omega)\cos(\theta)d\omega
$$
{% endraw %}

Double dollar block in raw as a separate paragraph:

{% raw %}
$$
\int_{S^2}f_s(\omega)\cos(\theta)d\omega
$$
{% endraw %}