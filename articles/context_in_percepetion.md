---
layout: post
title: Context & Perception with Computer Vision
---

*September 2024*

When my family or friends ask what I do, I say that computer vision automates visual perception. This of course is recieved with the same amount of confusion as any other explanation. However, the point is valid and is a solid reminder that like many fields related to AI/Machine Learning, computer vision is rooted in lessons learned from our brains. 

<p>&nbsp;</p>

### Context in Human Perception
Psychologists would be mad at me if I weighed in on the debate between the distinction between [cognition vs. perception](https://www.psychologicalscience.org/observer/cognition-and-perception-is-there-really-a-distinction) or even just [top-down vs. bottom-up perception](https://jackwestin.com/resources/mcat-content/perception/bottom-up-and-top-down-processing#:~:text=%E2%80%A2-,Perception%20refers%20to%20the%20way%20sensory%20information%20is%20organized%2C%20interpreted,(top%2Ddown%20processing).). It's incredibly cool stuff to read about it and if you fancy yourself an "automater of perception" it's worth a few minutes of your time. TLDR - everyone can agree that stuff you already know helps you percieve new stuff correctly.

<p>&nbsp;</p>

### Context in Computer Vision
As mentioned a few hundred pixels above, computer vision engineers automate visual tasks: automating infrastructure inspections, automating the process of reading license plates, etc. While customers may ask for 99.9% accuracy, modern computer vision models when deployed never quite reach that standard. In production, achieving high accuracy isn’t solely about refining the model itself; it's also about understanding the context in which the model operates.

By now, you're probably thinking about something like object tracking. Object tracking is one way to provide context. Tracking an object across frames ensures that the system doesn’t treat each detection as an isolated event but instead connects it to a broader sequence of information. However, tracking alone isn't always sufficient. Object tracking is like when a baby gains object permanence: the model knows it's the same ball, but that is the extent of its understanding.

A more robust approach involves building a post-processing backend that not only learns from what the system has seen but actively uses that knowledge to refine future detections. By storing and analyzing previous detection data, the system gains the ability to compare new inputs to past observations, enhancing its accuracy over time. In the same way engineers who take advantage of sensor fusion untangle meaning from multiple modalities, known data such as weight, time, size, speed, etc., can additionally be included in the detection refinement process. Essentially, the model begins to "learn" the environment in which it's deployed, adapting to the nuances of its surroundings.

This ongoing learning and refinement improves the overall performance of the system beyond what real-time detection alone can achieve. It's the computer equivalent of context in human perception—previous knowledge informs new understanding, leading to more reliable, context-aware detections.

By now, hopefully, the deployment of multi-modal models has come to mind. While outside the scope of this article, it's something I know from personal experience has tremendous potential. Feel free to reach out if it's something your company needs assistance with.