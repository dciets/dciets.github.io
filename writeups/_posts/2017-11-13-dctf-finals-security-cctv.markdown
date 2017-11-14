---
layout: post
ctf: "DCTF Finals 2017"
title: "DCTF Finals 2017 - Misc - Security CCTV"
---

We are presented with a website that ask to "Enter your authentification token!" and are given a pretty large image of a desk.

<img src="https://i.imgur.com/yrcylfS.png">

I remember that in the challenge description mentionned something along the lines that we should be able to find the authentification token. Observing the image a bit further I notice a QR code on the smartphone along with its reflection.

<img src="https://i.imgur.com/CV3y8G4.png">

At this point, I thought I solved the challenge and began playing in Photoshop to fix and read the QR code. This turned out to be a waste of time since the QR code changes about every 30 seconds.

Moving on, I noticed there’s an area of gray pixels that perfectly fits the QR code. This probably means that the image is generated without much processing to account for shadows for example. The perspective is not very natural either, you can see that the QR code is not fitting well the border of the right side of the smartphone. This means it should be relatively easy to script it.

Luckily, I’ve had a bit of experience with OpenCV on a <a href="https://github.com/clubcapra/Ibex/blob/master/src/capra_filters/scripts/perspective_calibration.py#L69-L100">robotic project to automatically remove the perspective distortion of a camera</a> and the task turns out to be similar.

The first step is to find a homography between the QR code on the camera image and a target square image then warp the image such that the QR code fits the given square. The same process is applied to the reflection on the laptop. This method is simple to use since it only requires a mapping of points on the image to points on the target square. Then, OpenCV does some magic maths to figure out how to transform the image.

When those images are composed, we have a nearly complete QR code but there’s is still one thing missing. There’s an alignment tracker hidden underneath the tissue that prevents most QR code reader from successfully reading the code. This tracker is added over the composed images approximately where it should be. QR codes have a lot of error correction embedded in them. Even though some parts of missing, it doesn’t prevent QR code reader from reading it.

Here is the hackish script used to solve the challenge <a href="https://gist.github.com/Becojo/37f7f47b27b532f0353a7235f42b73c6">https://gist.github.com/Becojo/37f7f47b27b532f0353a7235f42b73c6</a>.

Visually, this is what the process looks like

<img src="https://i.imgur.com/h5HSq1u.png">

---

Becojo - <a href="https://northerncoalition.github.io/">NorthernCoalition</a>
