[![Build Status](https://travis-ci.org/japaric/criterion.rs.svg?branch=master)](https://travis-ci.org/japaric/criterion.rs)

# criterion.rs

This is a port (with a few modifications) of
[Haskell "criterion" benchmarking library](http://www.serpentine.com/blog/2009/09/29/criterion-a-new-benchmarking-library-for-haskell)
to Rust.

Addresses [mozilla/rust#6812](https://github.com/mozilla/rust/issues/6812) and
I hope it'll help with
[mozilla/rust#7532](https://github.com/mozilla/rust/issues/7532)

I encourage you to look at this
[braindump](http://japaric.github.io/criterion-braindump), for an explanation
(with plots!) of how criterion works.

## Run the examples

```
$ make && make test
estimating the cost of precise_time_ns()
> mean:    17.281 ns
> median:  17.270 ns
> std dev: 55.225 ps

benchmarking fib_5
> collecting 100 measurements, 524288 iters each in estimated 1.8906 s
> found 4 outliers among 100 measurements (4.00%)
  > 3 (3.00%) high mild
  > 1 (1.00%) high severe
> bootstrapping sample with 100000 resamples
  > mean:    36.430 ns ± 6.5817 ps [36.419 ns 36.444 ns] 95% CI
  > median:  36.420 ns ± 3.6513 ps [36.412 ns 36.421 ns] 95% CI
  > std_dev: 65.780 ps ± 6.0348 ps [53.181 ps 76.776 ps] 95% CI

benchmarking fib_10
> collecting 100 measurements, 32768 iters each in estimated 1.4569 s
> found 4 outliers among 100 measurements (4.00%)
  > 2 (2.00%) high mild
  > 2 (2.00%) high severe
> bootstrapping sample with 100000 resamples
  > mean:    440.32 ns ± 94.507 ps [440.14 ns 440.51 ns] 95% CI
  > median:  440.06 ns ± 254.11 ps [439.65 ns 440.37 ns] 95% CI
  > std_dev: 935.46 ps ± 117.37 ps [707.91 ps 1.1652 ns] 95% CI
(...)
```

## Done so far

* Estimation of the cost of reading the clock (`precise_time_ns()`)
* Outlier classification using the box plot method (IQR criteria)
* Removal of severe outliers (this is **not** done in the original criterion)
* Bootstrapping: point estimate, standard error and confidence interval
* Convert to library
* Bencher-like interface
* Bencher configuration
* Benchmark groups
* Some examples

## Not (yet?) ported from the original

* outlierVariance, this method computes the influence of the outliers on the
  variance of the sample
  * this still looks too magical to me, using only the sample size, and the
    point estimates of the mean and the standard deviation, the author
    classifies the effect of the outliers on the sample variance
    * there are no references of the method used to do this
  * some rough ideas that might accomplish this:
    * the SEM (standard error of the mean) is the variance of the population
      over the square root of the sample size, I could compute the variance of
      the population and compare it against the bootstrapped variance.
    * Fit the bootstrapped distribution to a normal distribution, and look at
      the R squared.
    * Look at the skewness of the bootstrap distributions.

## TODO

* More testing
* Compare the results generated by criterion.rs with the results generated by
  Rust Bencher algorithm
* Compare the current basic bootstrap against the BCa (bias corrected and
  accelerated) bootstrap
* Save metrics to json file
* Hypothesis testing
  * execution time improved or regressed?
* Check if the sample is garbage
  * may be caused by CPU throttling or CPU usage peaks
    * should translate into high variance in the sample
  * background constant CPU usage should be hard to detect
    * this affects more the mean than the variance
* Documentation

# Wishlist

* Plot the [PDF](http://en.wikipedia.org/wiki/Probability_density_function) of
  the sample
  * computing the PDF is expensive
  * PDF from the sample is not too reliable, a PDF from the bootstrap would be
    better, but that would be even more expensive
  * need plotting library
    * gnuplot? is the license compatible with Apache/MIT?
* Interface to benchmark external programs (written in other languages)
  * Addresses the last point in
    [mozilla/rust#7532](https://github.com/mozilla/rust/issues/7532)
  * Something like [eulermark.rs](https://github.com/japaric/eulermark.rs)
    * See eulermark results [here](http://japaric.github.io/eulermark.rs)

## Unresolved questions

* Is sensible to remove the severe outliers in **all** the cases?
  * Removing outliers will always reduce the variance in the sample
* Can we continuously remove the severe outliers from the sample, until the box
  plot analysis yields no more severe outliers?
* When performing several benchmarks, heavy benchmark may affect the benchmarks
  that follow (hot CPU?), how do we address this?
  * Add a cooldown time between benchmarks?

## License

criterion.rs is dual licensed under the Apache 2.0 license and the MIT license.

See LICENSE-APACHE and LICENSE-MIT for more details.