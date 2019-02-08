---
layout: post
title: A lesson in product design and UI/UX
author: Manoj Awasthi
categories: product, design,tech
---

Today, an otherwise rudimentary discussion at workplace ended up in a learning (a small one but definitely one that I'll remember forever) in product design or UI/UX. 

Following image was shared for a certain feedback: 

<img src="/public/assets/img/tk.jpeg" width="200" />
 
At this, I shared a feedback almost immediately:

> .. one feedback: `matched search text` should be bold rather than `non matched search text`.

I felt quite strongly about this since it felt so wrong to me. This was soon validated by other colleagues who +1'ed the suggestion. As an engineer who had the opportunity to work on regular expressions as well as more advanced search engines based on Lucene,  I know that when we search for a `keyword` in a `text` then `matched substring` has quite an importance. 

But then another colleague shared following screenshot: 

<img src="/public/assets/img/gl.jpeg" width="200" />

Idea is that if it is so wrong, why is Google (the behemoth search engine) doing this? It stuck to me that there must be some reasoning behind it and that's when I and a colleague came up with this reasoning independently (shared almost at the same time): 

> Possible reasoning: Goal is for the user to click on the suggested items so they should be the one being highlighted. Potential targets.

> Hypothesis on google's approach: considering you already know what you typed, the bold text is to attract attention to the suggestion.

Later in the wee hours I thought testing some other web properties on this behavior and found mixed results. Following are the examples where they do highlight the searched (matched string): 

<table>
	<tr> 
	<td>
		<img src="/public/assets/img/bl.jpeg" width="200" />
	</td>
	<td>
		<img src="/public/assets/img/lz.jpeg" width="200" />
	</td>
	<td>
		<img src="/public/assets/img/fl.png" width="200" />
	</td>
	</tr>
</table> 

I take the liberty of calling this more of an Engineer driven approach. In this scenario, you basically limit yourself based on what the tool is providing you with (shares `matched string`) but in ideal scenario (again, taking liberty) you want user to follow certain paths based on what he or she has typed in and it is more important to highlight those parts (tool still helps since it is just inversion that need be done).

### Learning 

* Think critically but have deeper thoughts on the positives and negatives before jumping to conclusion.
* Think from the audience or customer point of view rather than from engineer point of view. 

Not really sure of the impact of the design choices here but in a world which is overflowing with information I believe such small design  considerations will help a lot. It's like guiding user to focus only on what he or she needs to.

