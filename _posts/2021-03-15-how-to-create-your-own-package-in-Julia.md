---
layout: post
title: "How to create your own package in Julia"
author: "Daniel Biro"
categories: julia
tags: [julia, package, github, create]
image: 2021-03-15-header-julia-package.png
toc: true
---

In this post, I will show you how to create your own Julia package in just a few easy steps. 
I have found the Julia programming language about a year ago and since then I am a huge fan. My thesis required a lot of time-consuming Monte Carlo simulations and originally the code was written in MatLab and it was just fine to finish my thesis but I still have plans to publish the study at some point. 
To achieve that goal I needed a more scalable solution, not just a simple script. 

My first idea was to re-implement everything in Python, however, I quickly realized that Python was not much faster and the way how NumPy handles matrices and vectors is just not for me. 
I always try to keep an open eye on exciting programming languages and when I heard that there is one out there that combines features from MatLab and Python and it is said to be super fast, I had to look into it. In my opinion, Julia has its issues and for most of my work I am still using Python as default, however, to implement my MCMC simulation-based models, Julia is just a really good choice. As my codebase grew, I started thinking about how could I convert it to a proper package type of project, and as it turned out, it is quite easy, you just have to follow these steps:
1. Open up Julia in terminal
2. Navigate to the location where you want to create your package (`cd("..")` and `pwd()` help)
3. Type `]`, this activates the Julia Package Manager
4. Run `generate YourPackageName` to generate your package
5. Run `cd("YourPackageName")`
6. Type `]` and run `activate .`, this will activate your package. Now the package is ready to use, but if you want to import other packages within your project, you have to add those by `]` and `add OtherPackages` (eg Distributions, LinearAlgebra, etc.)
7. `import` the package and run your implemented functions

![generate_package](/assets/img/2021-03-15-generate-package.png "Generate Julia package")

Tips and tricks, and some additional info:
- By default, the package contains an `src` folder that will have all your implemented codes
- The `Project.toml` and `Manifest.toml` files are responsible for handling the added external packages
- You can create [unit test](https://docs.julialang.org/en/v1/stdlib/Test/) for your package and store them in `test/runtests.jl`. Running the tests is very easy, type `]` and run `test YourPackageName`
- Julia recommends the UpperCamelCase formatting for package names
- You have to activate your package every time you launch a Julia session before importing, otherwise:<br>
![import_error](/assets/img/2021-03-15-import-error.png "Import Error")
- Add the Revise package to your project (`]` and `add Revise`). It is a very useful package during the development of your project. Before importing your package, load Revise `using Revise`. If you make changes to your code it will automatically use the updated version and you don't have to reload Julia
- You can also [include](https://docs.julialang.org/en/v1/manual/code-loading/) your package by `include("path_to_package\\YourPackageName\\src\\YourPackageName.jl")` and use it with `using .YourPackageName`
- Finally, your package can be deployed to the [General registry](https://github.com/JuliaRegistries/Registrator.jl?installation_id=13503700&setup_action=install#how-to-use) and everyone can add it as any other Julia package

### Useful resources:
- [Pkg.jl](https://julialang.github.io/Pkg.jl/v1/creating-packages/)
- [Developing your Julia package](https://medium.com/coffee-in-a-klein-bottle/developing-your-julia-package-682c1d309507)
- [How to start creating packages for Julia with Revise.jl](https://thibaut-deveraux.medium.com/how-to-start-creating-packages-for-julia-with-revise-jl-bdb47fd4ca5a)