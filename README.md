# IshgardCraftOptimizer

The plan for this project is simple: given an amount of Ishgard crafting materials, what is the most number of items you can make with them?

The other part of the plan for this project is to use it to learn Javascript. I want to make this tool easily accessible to everyone, which means making a website that people can use to input their materials inventory and get their own optimized solution. I decided I would build out the logic in C# first, the language I know best, to work out any issues I encountered before turning it into Javascript.

I have several pages of handwritten notes and analysis/calculations on my development of this tool, but they don't really provide much information without the proper context. I'm going to use the next few sections to explain my thinking and approaches to finding an optimizer for this problem. It's really the "how/why" of how I came to this solution. It's also here so if people find flaws with my logic, they can point out where and why and I can make corrections. If you just want to know what I used, skip to the end section.

# Quick overview of Final Fantasy XIV, Ishgard crafting, and Kupo of Fortune

If you are unfamiliar with Final Fantasy XIV, the second Final Fantasy MMO, I'm going to try to detail all the information pertinent to this tool here.

In FF XIV, there is an area of the game (Ishgard) that gets damaged in the first expansion, Heavensward. In the Shadowbringers expansion, the developers added a way for the crafting classes to contribute items to a restoration effort for this damaged area. To make these restoration items, players need materials gathered from a special area tht was added at the same time the restoration was implemented, specifically to support the restoration effort. When you donate a crafted item of **appropriate difficulty** for your crafting level, you are given a Kupo Stamp on a Kupo of Fortune card. Think of these as frequent customer cards at a restaurant. When a card has acquired 5 stamps, you can use it to play the Kupo of Fortune mingame, a luck-based scratch card game with exclusive rewards. Some of these are rare occurrences making them very valuable, either to be acquired or to be sold on the player market. This is why I decided to make this tool.

In FF XIV, there are 8 crafting classes: Carpenter (CRP), Blacksmith (BSM), Armorer (ARM), Goldsmith (GSM), Leatherworker (LTW), Weaver (WVR), Alchemist (ALC), and Culinarian (CUL). Crafting classes have levels just the same as combat classes do, which mostly comes into play in that Ishgard crafting recipes change as you move up through the level brackets. It should also be noted that you cannot just craft a bunch of level 20 stuff if you're level 80 and have it count for a Kupo Stamp, hence the emphasis on "appropriate difficulty" above. For a given level bracket, there are 10 kinds of materials provided by the special area to make Ishgard restoration items. Each crafter uses 3 of these materials from its level bracket. This means that crafters can and do share materials between them, prompting me to ask how a given dataset of materials can best be used to make the most number of restoration items, providing the player with the most number of Kupo Stamps. My work on this tool focused on the materials used by the level 80 crafts (level 80 being the current maximum level), but I abstracted things out enough that it should work for any level bracket as long as the specific level bracket crafter-material matrix is supplied.

# How I approached the problem

Inspired by the Kupo of Fortune, I decided that if I want to earn the most kupo stamps, I would need to optimize my material usage. So I set out to find/make an algorithm that would do just that. I had a few different initial approaches to the problem. Do I start with the material that has the highest count and work down? Do I start with the crafter that can make the most crafts and run it until materials are exhausted and repeat? Do I check the potential crafting count for each crafter, and then recurse through different branches if I hit a tie?

I stepped through each of these by hand at first, and I found that they all generally end up close to each other in terms of final craft count, but the one that consistently gave the best result was picking the crafter with the highest crafting count and running it to exhaustion, then repeating it.
While working through these different approaches at a higher level, I quickly realized something: I needed a way to handle tiebreaks.

# Tiebreaks and Priority

As before, I had a couple ideas for how to approach tiebreaks. Should I look at the materials involved in the crafters that are currently tied, picking the one with the most materials remaining? With the least materials remaining? Should I pick the crafter that impacts the fewest other crafters?
When I hit on that last idea, I decided to sketch out what the potential branches would be if I had exactly 1 craft's worth of each material, effectively finding all possible solutions to an 8-way tie. This is where I found something interesting and is what led me to create the priority system I use for tiebreaks.

Each crafter uses 3 different materials. That means that in a best case scenario, I was going to get 3 items (10 materials total, each craft uses 3 different ones, if the crafters don't interact, that means 9 of the 10 materials are used, 3 items with 1 material untouched). Because of how the materials are used in the crafter recipes, some of the crafters impact a LOT of other crafter. For example, of the 8 crafters, WVR impacted the most at 5 crafters besides itself. That meant there were 2 crafters untouched when 1 WVR item was made. And the remaining 2 crafters impacted each other. This meant that picking WVR first resulted in 2 potential paths, both yielding 2 items at best. WVR was unequivocally the worst choice.

But it's not enough to say that I should prioritize based only off the items a crafting path would produce, most paths DO yield 3 items, not 2. So I decided to look at how many paths a specific crafter would create when used as the starting point instead. This showed a bigger variance with WVR providing the fewest paths, 2, and ARM, ALC, and CUL providing the most, 5. When I got to this point, I decided to also look at how many paths yielded how many items, in an attempt to tiebreak priority, but the numbers came out the same regardless. In the case of a tiebreak at the same priority, I decided to either pick a craft that was already used, or just let the first one selected by the tiebreak. Using the number of paths a given crafter would provide as a priority, with higher numbers being better, I was reasonably confident I could deal with most tiebreak scenarios.

# Selecting an approach

One of the approaches had tiebreaking already somewhat baked in, the one that would recurse through different branches when a tie was encountered. I worked through a couple of these scenarios and quickly realized just how many potential branches this could yield, especially at higher input material counts. I tried a few datasets at lower material counts and encountered exactly what I feared: one of the branches yielded more items than the other, but it didn't come from the priority system I had laid out. I discovered why and because I didn't have a good idea how to detect those scenarios, I decided to run down another approach instead.

I chose to run down the "crafting to exhaustion" approach next. It was simple enough to calculate, both by hand and in code, but I didn't want to do it first BECAUSE of the simplicity. I wanted to see the results of in-depth approaches first as it seemed to me checking the state of things after every craft would yield the best results. So I ran the same datasets through this version of the algorithm (using the same priority system for tiebreaks) and I got the same best-case answer as the "recursing" approach every time. I was a little surprised, but as I thought more about it, I realized it made sense. It's intuitively correct to pick the crafter that has the most crafts available, the only missing piece was how to deal with tiebreaks. And because my priority system is based on how many possible crafting paths a starting craft can provide, it meshes perfectly with an approach that will exhaust the materials for a selected crafter each time. I had found the approach I would use for this tool.

# Conclusion/Summation

In the end, one of the first solutions I thought of turned out to be correct: picking the crafter that can craft the most items, crafting it until the materials are exhausted, then again picking the crafter that can craft the most with the updated materials list. While this will yield nearly-optimal results in almost all cases, the addition of my priority system to handle tiebreaks is what makes it give the best answer each time.

I was very happy the answer turned out to be one of simpler approaches as it is much easier to code, test, and debug!
