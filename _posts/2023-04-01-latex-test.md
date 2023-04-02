---
layout: post
title:  "LaTeX Test"
date:   2023-04-01 19:03:25 +0900
categories: jekyll update
---
$\sum_{i=1}^{n}f(x_i)$ You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the $(a+b)^2 = a^2 + 2ab + b^2$ 이고 $(a-b)^2 = a^2 - 2ab + b^2$ 이므로 $a^2 + b^2 = (a+b)^2 - 2ab = (a-b)^2 + 2ab$ 인데 아래와 같이 인라인 수식이 길어져도 $\lim_{x\to\infty}a_n = L \require{mathtools} \enspace \xLeftrightarrow[]{\text{definition}} \enspace \forall \epsilon \gt 0 \enspace \exists N \in \mathbb{N} \enspace \text{such that} \enspace n \gt N \implies \left\lvert a_n - L \right\rvert \lt \epsilon$ 이렇게 수식 밑에 스크롤바가 생기며 레이아웃이 깨지지 않는다 site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

$$ x^2 + \frac{1}{x^2} = \left(x + \frac{1}{x}\right)^2 - 2 = \left(x - \frac{1}{x}\right)^2 + 2 \quad \sum_{i=1}^{n}f(x_i) = f(x_1 ) + \cdots + f(x_n )$$

Jekyll requires blog post files to be named according to the following format:

`YEAR-MONTH-DAY-title.MARKUP`

Where `YEAR` is a four-digit number, `MONTH` and `DAY` are both two-digit numbers, and `MARKUP` is the file extension representing the format used in the file. After that, include the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight ruby %}
def print_hi(name)
  puts "Hi, #{name}"
end
print_hi('Tom')
#=> prints 'Hi, Tom' to STDOUT.
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
