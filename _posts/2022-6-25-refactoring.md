Blunder 8.0.0
-------------

It's been a while since I've had the chance to work on Blunder or update this blog. But having wrapped up
my freshman year of college about a month ago, I've had time this summer to begin working on Blunder again,
and now most recently, to come here and update the blog.

The next release of Blunder will be 8.0.0, and will be near the one year anniversary of the first release of Blunder. For this
release, in similar fashion to 5.0.0, I started from a pretty major refactoring of the codebase, cleaning-up code, documentation,
and optimizing, which contributed to Blunder 8.0.0 being a good bit faster then 7.6.0. 

Once the basic skelton of Blunder was back in place, I started adding back in features from 7.6.0, making
sure to test them more consistently. Some ideas from 7.6.0 stuck, others didn't and were discarded, but
overall in the end I was happy with the resulting code base, and I eventually achieved equality with 7.6.0's
strength level. The search and evaluation is a good bit different from the orginal code in 7.6.0, but I think
it's cleaner and better written.

An important factor in Blunder 8.0.0's quick rise in strength again is the 
[new gradient descent based texel tuner I wrote](https://github.com/algerbrex/blunder/blob/develop/tuner/tuner.go). The code isn't the cleanest,
but it's worked quite well and has allowed me to very quickly tune Blunder 8.0.0's evaluation to the same level that took me several months to reach
with Blunder 7.6.0's naieve texel tuner. I later updated the tuner to use AdaGrad instead of vanillia gradient descent, which made the number of epochs
necessary to reach good convernge higher, but made the overall optimizing smoother and more consistent, leading to better tuned values.

The idea of gradient descent is actually quite a straightfoward idea; the hard part was deriving the correct gradient formula to use, and then
implementing that efficently in code. In the coming weeks I plan on covering the process in more detail on this blog, as well as writing a dedicated
paper on how I set up the tuner and derivied the correct gradient formula. Hopefully such an article will be useful to those who were very new to the
whole process like I was.

Once I reached strength equality with 7.6.0, I started on tweaking the search and evaluation, as well as reviewing new features, to find a strength gain.
It's been a slow process, but I've made some decent progress, and the current dev version of Blunder is about ~30 Elo stronger then 7.6.0 from the latest
tests I ran. This was at a bullet time control of 10 seconds with a 0.1 second increment, so hopefully the gain scales better at longer time controls. 
The biggest contributors to this gain, besides the refactoring outlined above, were some tweaks to king safety scoring, 
adding [safe-mobility](https://www.chessprogramming.org/Mobility#Safe_Mobility) for knights, implementing internal-iterative deepending, and adding 
knowledge of commonly drawn, or drawish, endgames to Blunder's evaluation.

Though it's not the strength gain I hoped for, it's still solid progress in my estimation, considering the limitations of my time, hardware, and programming
knowledge. It's a commonly known phenomena that the stronger an engine gets, the harder it can become to consistenly make progress, as many of the "obvious"
improvments have already been implemented, meaning it takes many more hours of indepdent testing and experimenting to find progress. But of course, I find this
to be the most rewarding period of an engines development, despite it's difficulties.

I'll probably be releasing Blunder 8.0.0 pretty soon, depending on whether or not the next few ideas I have stick or not. I've also been made aware of some
bugs still lurking within the bowels of the latest release of Blunder, and I'd like to try to squash them before this next release, although I'm not sure
how feasible that is given the limited knowledge I have right now of their occurances.

Lastly, a large part of this refactoring was prompted by the release of Go 1.18, which finally added support for generics to the language, meaning I could clean-up
many parts of the code base that repeated logic but for different types. But even more interestingly for me, the release of 1.18 came with the ability to now
[target AMD64 microarchitectures](https://utcc.utoronto.ca/~cks/space/blog/programming/GoAmd64ArchitectureLevels), meaning that compiles can now be made the 
target specfic extended instruction sets such as AVX or BMI1. This could mean I might be able to produce faster versions of Blunder, although I have yet to
fully experiment with the new feature. Nevertheless when I release Blunder 8.0.0, I'll likely provide some pre-built compiles for a couple of microarchitectures.
