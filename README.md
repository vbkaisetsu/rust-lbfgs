
# LBFGS

This repo contains my ongoing efforts to port Naoaki Okazaki's C library
[libLBFGS](http://chokkan.org/software/liblbfgs/) to Rust. Check [rust-liblbfgs](https://github.com/ybyygu/rust-liblbfgs) for a working wrapper around the original
C codes.

[![Build Status](https://travis-ci.org/ybyygu/gchemol.svg?branch=master)](https://travis-ci.org/ybyygu/gchemol)
[![GPL3 licensed](https://img.shields.io/badge/license-GPL3-blue.svg)](./LICENSE)


# Motivation

-   Bring native LBFGS implementation to Rust community.
-   Learn how a great optimization algorithm is implemented in real world.
-   Learn how to "replace the jet engine while still flying" [URL](http://jensimmons.com/post/jan-4-2017/replacing-jet-engine-while-still-flying)
-   Make it more maintainable with Rust high level abstraction.
-   Improve it to meet my needs in computational chemistry.


# Todo

-   [ ] add option to disable line search for gradient only optimization
-   [ ] Parallel with rayon
-   [ ] SIMD support
-   [X] Fix issues inherited from liblbfgs [URL](https://github.com/chokkan/liblbfgs/pulls)


# Features

-   Clean and safe Rust implementation.
-   OWL-QN algorithm.
-   Closure based callback interfaces.


# Usage

    // 0. Import the lib
    use lbfgs::lbfgs;
    
    // 1. Initialize data
    let mut x = [0.0 as f64; N];
    for i in (0..N).step_by(2) {
        x[i] = -1.2;
        x[i + 1] = 1.0;
    }
    
    // 2. Defining how to evaluate function and gradient
    let evaluate = |x: &[f64], gx: &mut [f64]| {
        let n = x.len();
    
        let mut fx = 0.0;
        for i in (0..n).step_by(2) {
            let t1 = 1.0 - x[i];
            let t2 = 10.0 * (x[i + 1] - x[i] * x[i]);
            gx[i + 1] = 20.0 * t2;
            gx[i] = -2.0 * (x[i] * gx[i + 1] + t1);
            fx += t1 * t1 + t2 * t2;
        }
    
        Ok(fx)
    };
    
    // 3. Carry out LBFGS optimization
    let fx = lbfgs()
        .with_max_iterations(5)
        .minimize(&mut x, evaluate)
        .expect("lbfgs run");
    
    println!("fx = {:}", fx);

The callback functions are native Rust FnMut closures, possible to
capture/change variables in the environment.

Full codes with comments are available in examples/sample.rs.

Run the example:

    cargo run --example sample

