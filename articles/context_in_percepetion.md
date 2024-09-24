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
As mentioned a few hundred pixels above, computer vision engineers automate visual tasks: automating infrastructure inspections, automating the process of reading license plates, etc. While customers may ask for 99.9% accuracy, modern computer vision models when deployed never quite reach that standard. In production, achieving high accuracy isn’t solely about refining the model itself; it's also about understanding the context in which the model operates, or using stuff already known to percieve new stuff correctly. 

By now, you're probably thinking about something like object tracking. Object tracking is one way to provide context. Tracking an object across frames ensures that the system doesn’t treat each detection as an isolated event but instead connects it to a broader sequence of information. However, tracking alone isn't always enough. Object tracking is comparible to when a baby gains object permanence: the model knows it's the same ball, but that is the extent of its understanding.

A more robust approach involves building a post-processing backend that not only learns from what the system has seen but actively uses that knowledge to refine future detections. By storing and analyzing previous detection data, the system gains the ability to compare new inputs to past observations, enhancing its accuracy over time. In the same way engineers who take advantage of sensor fusion untangle meaning from multiple modalities, known data such as weight, time, size, speed, etc., can additionally be included in the detection refinement process. Essentially, the model begins to "learn" the environment in which it's deployed, adapting to the nuances of its surroundings.

### A Very Basic Example
Imagine a parking garage. The goal is to know which cars are entering and exiting this parking garage. To do so, a good approach would be to place cameras at the entrances and exits so they can capture the license plates of each vehicle as they come and go. A model must be trained to read the license plate of each vehicle, [a task very well studied in computer vision](https://en.wikipedia.org/wiki/Automatic_number-plate_recognition). After much work, your model has 99% accuracy when deployed. This means that in a parking garage that sees 2,000 unique vehicles per day (assuming each vehicle that enters also exits on the same day), the system would make 4,000 detections. At 99% accuracy, 40 detections would still be incorrect, an unacceptable amount—especially if this information is used to charge customers for their time in the garage.

Instead of accepting the output of each raw license plate reading as truth, a few contextual operations can be performed to improve accuracy:

1. **Compare vehicle exit readings to entrance readings**: If there is no match, look for candidates based on string similarity using algorithms such as Jaro–Winkler.
2. **Handle returning vehicles**: In situations where certain vehicles return regularly, look further back into historical detections to match new detections with those the system expects. Analyzing when these known vehicles most often enter and exit can further increase the accuracy of contextual corrections.
3. **Use visual context**: Is the vehicle’s make, model, and color for a given reading the same as that seen in historical detections? When comparing exit readings to entrance readings, is the same true?
4. **Analyze patterns of error**: What time of day, vehicle type, or character combinations does the system get wrong the most? This information could not only help identify out-of-distribution data, but also inform dynamic inference parameters, such as optimal IOU (Intersection over Union) or confidence thresholds.


### In Conclusion
This ongoing learning and refinement improves the overall performance of the system beyond what real-time detection alone can achieve. It's the computer equivalent of context in human perception—previous knowledge informs new understanding, leading to more reliable, context-aware detections.

By now, hopefully, the deployment of multi-modal models has come to mind. While outside the scope of this specific thought I had, it's something I know from personal experience has tremendous potential. Feel free to reach out if it's something your company needs assistance with.