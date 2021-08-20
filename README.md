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

# How I solved it

My initial implementation took the approach of running the crafter with the most counts to exhaustion, repeating that until the materials were exhausted. However, I started running into edge cases with non-optimal results when I allowed the usage of fewer than every crafter. Since I wanted the user to be able to set which crafters were available for use, I had to try a different approach.

My second implementation was to loop through all the available crafters, picking one as the starting crafter, have it use all the materials in the inventory, then it would recursively check all the remaining crafters left in the same fashion. It then saves the "high score" (or in the case of a tie, the one that uses the fewest different crafters) for that starting crafter. Once all paths have been checked, it presents the list of paths that match the high score to the user.

# Conclusion/Summation

Unfortunately, my first attempt at the algorithm turned out to be suboptimal and I didn't discover this until it was almost done. When I was deciding on a new approach, I did test a recursive version that branched after every single craft. As I suspected, and as you likely expect at this point, that took a LONG time. In fact, I don't think I ever let a run finish it was taking so long. So that idea got tossed out and I chose the recursive-lite version of consuming all the materials for a given crafter. I'm much happier with this version of the algorithm anyway, and it let me set the code up in a way that made it easy to implement existing, and likely any future, level brackets.
