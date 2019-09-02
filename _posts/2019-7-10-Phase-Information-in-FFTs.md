---
layout:     post
title:      Christmas in July - Hidden information in FFTs
date:       2019-7-10 12:00:00
summary:    Extracting information from the complex nature of Fourier Transforms and the application in research.
categories: Research
---

Check out the paper here: [https://arxiv.org/abs/1903.00515](https://arxiv.org/abs/1903.00515)

In my research, we image surfaces of materials with atomic resolution. A lot of the work deals with identifying patterns within these images, whether it's the atoms within a lattice or electronic standing waves. The best way look at these patterns is with Fourier transforms. The basic idea of a Fourier transform is that a function (or in this case an image) can be represented by a sum of sinusoidal functions with varying amplitude and frequency. For a continuous function this looks like and integral, but discrete data uses a sum. I'll consider a 1D discrete function \\(I(\vec{r})\\), but this is easily generalized for an N-dimensional function/dataset. If you want to do it in N dimensions, we just need N sums or N integrals.

\\[I(\vec{r}) = \sum_{\vec{k}} A_k e^{i \vec{k} \cdot \vec{r}} \\]

This notation is a bit specific to my scientific work. \\(\vec{k}\\) is usually associated with a crystalline momentum and can also be written as \\(\frac{2 \pi}{\lambda}\\) where \\(\lambda\\) is the wavelength of the periodic modulation within the data. \\(A_k\\) represents how strong the signal is at a given periodicity/wavelength.

A key component of this representation is that a function can be *transformed* one to one from "real space" to "reciprocal space" (or in my work "k-space"). Reciprocal space means that same function \\(I(\vec{r})\\), but now represented in terms of \\(\vec{k}\\) instead of \\(\vec{r}\\).  Transforming to reciprocal space changes the function from one in \\(\vec{r}\\) to a related function in \\(\vec{k}\\). A peak within \\(\tilde{I}(\vec{k})\\) would indicate that the original function \\(I(\vec{r})\\) has a strong periodic signal at the wavelength of peak location. The integral for this in a 1D discrete data set would look like:

\\[\tilde{I}(\vec{k}) = \sum_{\vec{r}} A_r e^{i \vec{k} \cdot \vec{r}} \\]

In this representation, all the information of the function is contained within the complex (as in complex numbers) value of \\(A_k\\) or \\(A_r\\). A very common data analysis step within research is to transform data from real space or time, to this reciprocal space or inverse time space. Most of the time, we care about where peaks are in reciprocal space or, other words, what are the dominant periodicities within an image. For example, if you look at the Google trends for "All I Want For Christmas" you will see strong peaks at multiples of 1 year. This is due to Christmas being an annual holiday.

However, there is more information in the Fourier transform than the amplitude of the peaks. Since it is complex, there is also a phase. But what is it good for? For this let us consider *Christmas in July*.

### Christmas in July

The Google trends for "All I Want for Christmas" can be easily downloaded into a csv, which stores the strength of that search for a given month since 2004. We can look at the trend in time and see very strong peaks separated evenly in time. This is of course due to the fact that Christmas comes but once a year. Taking the Fourier transform shows peaks at every integer multiple of a 12 month frequency. This is because if it is Christmas, then it was Christmas two years ago and three years ago and so on. The strong peak at zero is low frequency noise that is common in Fourier transforms of real data.

<p align="center">
<img src="/_img/google_trends_1.png" alt="drawing" width="600" />
</p>
<p align="center">
<img src="/_img/ft_1.png" alt="drawing" width="600" />
</p>

But what would happen if every decided to switch Christmas to July in 2011? The time before 2011 would have the same 1 year frequency as the years following, but for the year of the switch the distance between peaks would be halved. How would this show up in the Fourier transform? If we think about this in term of Sine waves, the time before and after the switch to July would have the same frequency and different phases. But in a Fourier transform, one frequency can only have one phase. Therefore, frequencies adjacent to the fundamental frequency need to cancel out in such a way to create the real signal. So if we look at the Fourier transform if we had this switch, we see that some of the peaks are split. Namely the odd peaks. This is because the canceling needed can only be done between odd multiples of the base frequency.

<p align="center">
<img src="/_img/google_trends_2.png" alt="drawing" width="600" />
</p>
<p align="center">
<img src="/_img/ft_2.png" alt="drawing" width="600" />
</p>

Within this clean data, it is easy to see the difference before and after the switch. But if the dataset is more complicated or noisy, how would we determine that a shift occurred? And by how much?

The best way to do this is through the phase of the Fourier transform. If we model this fundamental frequency as something that could change in time, i.e. \\[\vec{k} + \delta k(t)\\], then by figuring out \\(\delta k(t)\\) we can quantify the change of Christmas from December to July. If we look at the region of the first peak we see that it is split as expected. To isolate the \\(\delta k(t)\\), we can set all entries outside a small region near the peak to zero (apply a mask) and shift the non-zero entries to the center of the Fourier transform. This shift gets rid of \\(\vec{k}\\) leaving just \\(\delta k(t)\\). Bu t, we want to see what is changing in time not inverse time. If we take the Fourier transform again, (the inverse Fourier Transform), we are left with a real space image represented by:

\\[A_{\textrm{one year}} e^{i 2\pi \delta k_{\textrm{one year}}(t) \cdot \textrm{one year}}\\]

We can take the phase of this real space image to get \\( 2\pi \delta k_{\textrm{one year}}(t) \cdot \textrm{one year} \\). We would expect to see that the total shift is half the fundamental frequency of one year since we shifted Christmas from the 12th month to the 6th month. This would be associated with a phase difference of \\(\pi\\). We see below that there is a phase difference of about \\(\pi\\) associated with exactly the same time that the switch was made!


<p align="center">
<img src="/_img/peak_zoom.png" alt="drawing" width="600" />
</p>
<p align="center">
<img src="/_img/phase.png" alt="drawing" width="600" />
</p>
### Phase in Research

I used this same approach recently in research to determine the shift of atoms within a crystal lattice across a domain wall. For our explanation of the data, it was important that this shift be half a lattice constant, similar to the shift of Christmas half a year. Below you can see the original topography, a close of the split peak in the Fourier transform associated with a phase shift, and the "Phase map" which details how the two regions on each side of the domain wall are shifted relative to each other.

<figure>
<img src="/_img/phase_fig_2.png" alt="drawing" width="800" />
<figcaption>Using FFT phase information in research. a) topography of domain wall in FeSeTe. b) Close up of split Bragg Peak in FFT. c) Phase map showing a clear \pi phase shift </figcaption>
</figure>

If you want to read more about this research, check it out [here!](https://arxiv.org/abs/1903.00515) Thanks for reading!
