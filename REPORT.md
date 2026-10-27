# Assignment 2 Report


- **Name**: Marcela Mendes Gimenes Funabashi
- **Student ID**: 20251
- **Email**: mmendesgimen.ieu2022@student.ie.edu 
- **Group**: BBADBA 5A

## Business problem, and the cost of being wrong

During my summer internship at Brazil’s largest pharmaceutical company, I learned how important it is to detect manufacturing defects immediately and precisely. These laboratories produce at a very large scale to take advantage of economies of scale, with packaging lines running hundreds of blisters per minute. At the same time, they operate in one of the most critical and highly regulated industries in the country. This model supports one specific decision: whether each blister leaving the packaging line is intact and contains all of its pills, or whether it should be rejected. Blisters the model is confident are defective are automatically rejected from the line, and blisters it is unsure about are sent to a quality control operator for manual inspection. The model can be wrong in two directions, and with unequal magnitudes. A false positive (rejecting a blister that was actually fine) only costs the value of that blister and a few seconds of an operator’s time, which is a matter of cents. A false negative (letting a blister with missing pills reach the market) is a much more expensive mistake. A patient could take an incomplete treatment without noticing, the company could be forced into a batch recall, face sanctions from ANVISA and suffer lasting damage to its reputation and trust with pharmacies and doctors. Missing one defective blister can cost thousands of times more than wrongly rejecting a good one. For this reason, the model should be built to catch as many defective blisters as possible, even if that means rejecting or reviewing more good ones. Recall (sensitivity) on the defective class is therefore the metric this analysis should prioritize, and the decision threshold should be set low enough to keep missed defects close to zero, while keeping the number of blisters sent to manual review manageable for the operators.

## Dataset

To train and evaluate this model, I used the pill dataset published on Roboflow Universe by the workspace zhaosi-nicholas-z2xrh, project pill-j8vgy, version 63 (https://universe.roboflow.com/zhaosi-nicholas-z2xrh/pill-j8vgy/dataset/63), which is available under a CC BY 4.0 license. I chose this version because it is the largest one available, with 1,645 images (1,045 for training and 600 for validation), it applies no augmentation, and its only preprocessing step removes a class that drew a single box around the whole package, which is not needed for this decision. Each image shows one blister pack photographed inside a fixed inspection setup, with the same camera position, lighting and dark background. The pack is seen either from the front (the plastic cavity side) or from the back (the aluminium foil side). Every individual pocket is annotated with a bounding box and one of four classes: pill_front and nopill_front (a pocket with or without a pill, seen from the front), and pill_back and nopill_back (seen from the back, where the pill either still presses against intact foil or the seal is broken and the pill is gone). This structure translates directly into the decision described above: a blister is defective if at least one of its pockets is detected as nopill_front or nopill_back, and intact only if every pocket contains a pill, so the recall I prioritize is measured on these two “nopill” classes. This is what makes the dataset actually serve the problem rather than merely resemble it. Most pill datasets on Roboflow are built to identify which medicine a pill is or to count loose pills, while this one labels exactly the defect that leads to the expensive false negative, a missing pill inside a sealed blister. It also covers both sides of the pack, like real packaging lines do, and it was captured in a controlled setup that is much closer to an inline vision station than casual photos. Since the provided split has no test set, the 600 validation images should be divided in half, using one half to choose the decision threshold and keeping the other half untouched for the final evaluation, so the recall I report is not measured on the same images I used to tune it. The dataset still has limitations, such as that the images are still frames rather than footage from a line running hundreds of blisters per minute, and the empty back-side pockets show pills that were pushed out rather than never filled. For these reasons, the results should be read as a proof that the approach works, and a real deployment at a company would still require fine-tuning on images from its own packaging lines.

## What the dataset's composition implies

Version 63 contains 1,645 images and 21,794 annotated pockets. The training set has 1,045 images and the validation set has 600, with no test set provided. I will therefore split the 600 validation images at random into two halves of 300 images each, using a fixed random seed so the split can be reproduced. One half will be used to choose the threshold and the other will be kept untouched as the test set. The classes are very unevenly represented. In training, pill_front has 8,479 instances, nopill_front 2,821, pill_back 1,613 and nopill_back only 276. Most of the pockets are also small: a typical pocket covers only 0.4% to 0.7% of the image area, or about 6% to 8% of the image width. The smallest tenth cover less than 0.2%. The model will therefore have to detect small details in each frame, and empty pockets are among the hardest to see. Because my decision is made blister by blister, what really matters is how many defective blisters each split contains. The validation set has 368 defective and 232 intact blisters, so each half will hold around 185 defective blisters. Missing a single defect will then lower recall by about half a point. That is precise enough to compare thresholds on the front side of the pack. On the other hand, the back side has only 42 training images and 15 validation images containing a nopill_back pocket, which leaves around 7 or 8 per half. Recall on back-side defects would increase by more than 10 points with every miss, so I will not treat that number as reliable and will report it only for reference. The test set will also differ from what the line would really see in two important ways. First, 61% of the validation blisters are defective. On a real packaging line, the defect rate is a small fraction of one percent. This means I will be able to measure recall, but I will not be able to use these images to estimate how many good blisters would be rejected or sent to the operators for every defect caught. In production, that review load would be much higher than the test set suggests. What I therefore cannot conclude from my results is how the model would perform on a real production line. That includes its recall on back-side defects, the review workload it would create for the operators, and how well it would handle new products and packaging formats that it has not seen before.

## Finetuning and baselines

*Which model and task (detection or classification) and why. A results table on
the same test split: your finetuned model, the pretrained COCO model zero-shot,
and a trivial baseline. What changed, and whether the gap is as large as you
expected.*

## Per-class evaluation and failure modes

*Per-class AP50, or per-class precision/recall and the confusion matrix.
Which class carries the headline number, which one is dragging, and what the
model actually confuses with what. Show where it breaks, not just that it
works.*

## Cost matrix and operating point

*Your euro cost for each kind of error and where each figure came from. The
base rate you assumed. The cost-against-threshold sweep, and the operating
point you chose. State the assumption that would most change your answer if it
were wrong.*

## What a person still has to check

*Your two thresholds, and what share of cases they send to a person. How many
that is per day, how long one takes to check, and how many people that needs.
Is that realistic? Then your design priced against the three alternatives:
doing nothing, having a person check everything, and letting the model decide
on its own.*

## Limitations & next steps

*Real limitations you found, and concretely how you'd address each one with
more time or data. Not generic hedging.*

## Generative AI use disclosure

*Per the syllabus AI Policy: what you used and how, or "no AI content used."*
