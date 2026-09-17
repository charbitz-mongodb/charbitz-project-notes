# Title: A 1-5 Word Summary

* list of
* participants if
* applicable

## Summary

The summary is a one or two sentence description of what *engineering* solution is being supplied to a *technical* problem; this demonstration memo shows a general structure of a brief technical mini-design document.

## Background

Sometimes you need to explain a little background so that a technically-versed reader can understand the next section: the problem statement.

It is important to consider the following:

* bulleted lists are a good story-telling shortcut
* bullet points need not be complete sentences
* I perfer 3-7 words per bullet

One common struggle with Background statements is to avoid *assuming the problem statement* -- i.e. the background statement should demonstrate that the problem statement is relevant and real.

Sometimes it is helpful to think of a background statement as a description of symptoms (e.g. of an illness):

1. On occasion you have to explain a pain to a doctor.
1. Doctors need to understand what you were doing prior to onset.
1. The context of pain onset should be understood.
1. Otherwise, the doctor's expertise cannot make insights.
1. Without insights, the doctor is left guessing.
1. Rational decision making needs to minimize guess work.

Pictures and thousands of words:

* Sometimes you just need a quick diagram
* I like to draw my diagrams
* I then switch to the [Simple] text render mode in the left-hand pane
* Copy & Paste into markdown (use four leading spaces on the left)
* Markdown renderers will then use fixed-width font for your ASCII art!

The background section can also document constraints on the solution.  Constraints:

* Can be open, closed, or somewhere between
* Apply to specific scenarios
* Help the reader understand the trade-offs of the solution

## Problem

The problem statement explains *what* the **technical** problem *is* and not how the problem shows up -- that is for the Background section (where you might also find the business-oriented problem statement). Instead the problem statement is 1-3 sentences that clearly state what underlying issue needs a solution. Most commonly a problem statement needs to outline a choice that needs to be made.

## Decision evaluation criteria

Not all decisions require the same amount of analysis, it depends on how clear-cut the decision is. If there is enough uncertainty between the alternative solutions then the primary goal is to reduce the uncertainty.
One way is to first list the decision evaluation criteria which can be measures or requirements that are necessary to make a decision from one of the many solutions to the problem
* Criteria must be objective and ideally measurable
* If possible, give a rank to each criteria based on its importance
    * Mandatory criteria are higher ranked than nice-to-have criteria

Criteria can include
* Customer expectations and requirements
* Tech limitations
* Ownership and lifecycle costs of the chosen solution
* Impact to the timeline and delivery etc.

## Solutions

### Option 1: Generally prefer to outline options

Options are an excellent way to demonstrate what considerations you took into account when making a choice.

Sometimes it is helpful to include bulleted lists:

* I like to put my *considerations* into bullets.
* Other people prefer separate *Pros* and *Cons* lists.
* The key is clarity of thought.

### Option 2: Ignore options

While generally considered poor form, it is sometimes okay to not present a set of options.

This is sometimes the case when you quickly realize there is really only one reasonable course of action, but you must still demonstrate clear thinking; in this situation is lists of costs and consequences are helpful.

If you choose this option, the following are helpful:

* Action-items with explanation efforts.
* Consider describing the output artifacts.
* Name the concrete deliverable of the research.
* Do not: simply conclude that research must be done.
* Do: outline what you want the research to produce.
* Research can support/refute a hypothesis.
* A hypothesis should support predictions.
* Predictions should be made before experiments are run.

## Evaluation

Once you have one or more options listed above, an evaluation needs to be done to determine the recommended or chosen solution. In some cases this may seem straightforward and you do not need the same amount of analysis (likely as a result of some implicit evaluation method or subjective inputs from having domain knowledge).
* Regardless, a robust decision is one that is data driven while reducing the uncertainty and cost of changing the decision later in the project lifecycle.
* Document evaluation of the decision in all cases

Sometimes a conclusion is not clear, in such cases evaluate solutions against the defined criteria above and drive to a recommended solution
* This may be in the form of a table where the criteria are listed against the options
* One way to evaluate is by simply grading on which solutions meet which criteria which ones fall short(binary evaluation)
* Another option is to use weight to consider the higher ranked criteria with higher weights and compare

Examples

| Option | Concision | Easy Communication | Use-Case Driven |
|-|-|-|-|
| **Comparison Matrix** | Yes | Yes | Yes |
| **Multiple Bulleted Lists** | Yes | No | Sometimes | 
| **Free-form Paragraphs** | Unlikley | Occasionally | Unobviously | 


## Conclusion

Based on the above, you've arrived at a conclusion on the solution.
It is helpful to summarize which option you chose and why outside of the *short* Summary section above. One nice thing about a conclusion section is that it can convey you rationale. Colleagues can challenge that rationale in a code-review of the memo in the Conclusion section while leaving the summary largely unaffected.
* Document any dissent that arise around the conclusion and for the final option agreed upon document any risks and how we plan to manage the risk