---
layout: post
title:  "Welcome to Jekyll!"
date:   2016-07-27 12:31:14 +0000
categories: jekyll update
---
You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

{% highlight c %}
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>
#include <tests.h>
#include <utilities/benchmarking.h>
#include <utilities/logging.h>

// TODO: write a script that runs with and w/o prefetchers turned on. Fixes the
// cpu frequency to 2 GHz, check the function dump. Have it run in 4 modes:
// read, write, read, and possibly NTPS use quadwords instead of bytes?

#define repetitions_per_iteration 128
double timings[repetitions_per_iteration];
const size_t max_stride = 1 << 24;
const size_t min_buffer_size = (1 << 8);
const size_t max_buffer_size = (1 << 25);
buffer_type *buffer = NULL;

#define trash_size (1 << 26)
buffer_type cache_trash[trash_size];

int sort(const void *x, const void *y) {
  double xx = *(double *)x, yy = *(double *)y;
  if (xx < yy) {
    return -1;
  }
  if (xx > yy) {
    return 1;
  }
  return 0;
}

int main(int argc, char **argv) {
  buffer = valloc(sizeof(*buffer) * max_buffer_size);
  if (!buffer) {
    report(FAIL, "Couldn't allocate a buffer of size %lu", max_buffer_size);
    return -1;
  }
  for (size_t buffer_size = min_buffer_size; buffer_size <= max_buffer_size;
       buffer_size *= 2) {
    printf("size=%lu\n", buffer_size);
    for (size_t stride = 1;
         (stride <= max_stride) && (stride < buffer_size / 2); stride *= 2) {
      write_buffer(cache_trash, trash_size, 1);
      struct timespec T;
      write_buffer(buffer, buffer_size, stride);
      for (size_t u = 0; u < repetitions_per_iteration; u++) {
        tick(&T);
        for (size_t w = 0; w < stride; w++) {
          write_buffer(buffer, buffer_size, stride);
        }
        double time_taken = elapsed_since(&T);
        size_t n_access = buffer_size;
        timings[u] = time_taken / n_access;
      }
      qsort(timings, repetitions_per_iteration, sizeof(double), sort);
      printf("%lu %1.6e %1.6e %1.6e\n", stride, timings[0],
             timings[(repetitions_per_iteration + 1) / 2],
             timings[repetitions_per_iteration - 1]);
    }
    puts("\n");
  }
  return 0;
}
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
