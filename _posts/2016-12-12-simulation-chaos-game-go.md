---
layout: post
title: Simulation of chaos game in go 
author: Manoj Awasthi
categories: tech
---

Fractals are interesting. As happens with most things interesting in life I got introduced to fractals serendipitously while researching on another mathematical construct '[golden ratio](https://en.wikipedia.org/wiki/Golden_ratio)' which i needed to know about for implementation in an image editing feature I worked on as part of my day job few years back. 

[Fractals](https://en.wikipedia.org/wiki/Fractal) are self repeating patterns (either exact or approximately) and are found in nature aplenty (mostly the latter kind). One of the most intriguing ones are [Mandelbrot sets](https://books.google.co.id/books?id=0R2LkE3N7-oC&redir_esc=y) which have a mathematical connotation i.e. you can write a mathematical formula to describe mandelbrot sets - the fact that it involves complex numbers and the fact that this is found in nature is what makes them particularly interesting as fractals. This is when you start wondering if God is mathematician or if we discover mathematics rather than invent it. More on that later 

After reading through some of the articles on fractals online, I started reading a book on fractals recently. The first fractal it talks about is [sierpinski triangle](https://en.wikipedia.org/wiki/Sierpinski_triangle). So one sunday I spent implementing a simulation in go. 

![Sierpinski Triangle](/public/assets/img/st.png)

You can check the simulation in the form of GIF at this page: [https://awmanoj.github.io/public/sierpinski.html](https://awmanoj.github.io/public/sierpinski.html)

The code for the service generating the GIF is available at: [https://github.com/awmanoj/chaos](https://github.com/awmanoj/chaos)

If you have questions you can [reach out to me](https://twitter.com/awmanoj).
