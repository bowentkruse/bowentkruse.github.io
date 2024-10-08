---
layout: post
title: Interactive articles/tutorials on this site
---

# The Problem
You'll notice all of my project write-ups start with identifying the problem my solution solves. This write-up summarizes each of the interactive articles and tutorials available in the [articles](../../articles.md) section of this website, and so the problem being solved is a bit more intanbigle/conceptual. 

By making content interactive, it encourages a higher level understanding from the reader, and requires a higher level communication ability from the creator. Therefore, I consider the problem that interactive content solves to be communication shortfalls that are common in technical content. Whether that be from a lack of understanding/focus from the consumer, or a lack of ability to communicate from the creator. Communication is everything.

A secondary objective of this content is to fill-in any gaps I see in other publicly available content. Companies such as roboflow have enough content such that any diligent person with a basic programming ability could become a half-decent computer vision engineer. My goal is to provide original content focused on real-world use cases. 

<p>&nbsp;</p>

---

### [Quantifying Focus in Computer Vision Use Cases](../../articles/quantifying_focus/)

**Audience:** Computer Vision Engineers, especially those interacting with dataOps, developing sensors, or any data-centric developer.

**Topic:** Image clarity, or lack thereof, is critical for many reasons in any computer vision project. It's possible to quantify clearity.

**Interactive Content:**: The Colab Notebook downloads a dataset from roboflow, implements brenner's focus measure and 4 other variations, evaluates each variations effectiviness on the given dataset, and then simulates a threshold based filter (dropping blurry images). Success is measured by the number of blurry images the filtering logic can successfully identify and drop. 

**Conclusion:** Relative performance of a given focus measure is dependent on the dataset and use case. However, implementing focus measure operators is not a novel task and so finding an effective focus measure is closer to a routine experiment than novel task. 
