---
layout: single
classes: wide
title:  "[snn-#1] Radially Linked Network (RLN)"
last_modified_at: 2020-09-28
categories: research snn

author: Mihir Savadi
---

<div markdown="1" style="font-size:20px; font-family: Times New Roman">
This discussion is, in part, a follow up of [post [snn-#0]]({{site.baseurl}}/research/snn/Digital-Bidirectional-Leaky-Integrate-and-Fire/).The aim of the Radially Linked Network is to address resource related concerns of the DBLIF model were it to be placed in a conventional multi-layered perceptron, as well as provide an easily scalable multi-dimensional network for Spiking Neural Network (SNN) hardware implementations. The framework for the Radially Linked Network is explained first, and implications on DBLIF resource efficiency and other points are discussed further on. <br/>

Figure 1 below demonstrates the idea in a simple 2-dimensional form. <br/>

![Figure 1]({{site.baseurl}}/assets/research/snn1-fig1.png) <br/>
*Figure 1: 2D radially linked Neural Unit with growing radius*
{: .text-center}

The red squares in the figure represent a central neural unit, which receives inputs from its neighbouring blue neural units. The distance to which it connects to neighbouring neural units is denoted with the $$r$$ constant for ‘radius’. The resultant number of connections for each neuron in this 2D arrangement given a $$r$$ radius value is denoted by $$n$$ and is given by the following equation:
</br>

$$ n = (2⋅r + 1)^2 - 1 \tag{1} $$

Figure 1 and equation(1) demonstrated radial connections in the $$x$$ and $$y$$ axis’, but if we extend the same 
idea into the $$z$$ axis as well, the equation for $$n$$ becomes the following: <br/>

$$ n = (2⋅r + 1)^3 - 1 \tag{2} $$

The following equation for $$n$$ can be extrapolated from what we have so far, for any number of dimensions $$d$$: <br/>

$$ n = (2⋅r + 1)^d - 1 \tag{3} $$
where $$ r ≥ 0, d ≥ 1 $$
{: .text-center}

How would one fabricate an integrated circuit of a radially connected network in dimensions $$𝑑 > 3$$ ? Imagine Figure 2 below is a radially linked network of neurons with $$r = 5$$ and $$d = 3$$. Each of these neurons is assigned a coordinate, $$(𝑖, 𝑗, 𝑘)$$ with respect to an origin located at any arbitrary corner of the grid. <br/>

![Figure 2]({{site.baseurl}}/assets/research/snn1-fig2.png) <br/>
*Figure 2: Radially Linked network of r=5 & d=3*
{: .text-center}

Keeping $$r=5$$, but increasing to $$d=4$$, we would have equivalent 3-dimensional grids placed in positions relative to the original grid in a manner similar to how individual neural units are arranged in a singular 3-dimensional grid of $$r=1$$, sort of like a ‘grid of grids’. Each neural unit in each of these sets of grids would then receive an additional parameter in its coordinate – $$(𝑖, 𝑗, 𝑘, 𝑎)$$, where $$a$$ would range from $$0$$ to $$26$$, because in a radial network of $$d=3$$ and $$r=1$$, $$n = 26$$ according to equation(3). Each $$(I, J, K)^{th}$$ neuron in each grid would share the inputs of a neuron in every other grid with the same $$(I, J, K)$$ coordinates, with $$I$$, $$J$$, and $$K$$ being specific values in the $$x$$, $$y$$, $$z$$ coordinate space of each of the constituent $$d=3$$ grids. If we wanted a 5-dimensional grid and so on, we would further the arrangement similarly – adding an extra coordinate parameter, and linking every neuron with equal preceding parameters together as such. These multidimensional networks can then be built in 3-dimensional space. This also allows easy identification of each and every neuron. In a complex enough VLSI implementation, each neuron’s memory/parameters can be located and accessed for modification if that is desired, no matter the chosen value for $$d$$. <br/>

In an RLN, no matter the value of $$r$$, as long as it is greater than $$0$$, every single neuron will have implicit connection with and therefore an influence on every other neuron in the network, thereby providing a comprehensive and scalable network structure. Each neuron would only require a fixed number of weights, independent of the size of the network, equal to $$n$$ given by equation(3). If we carefully vary $$r$$, we can have direct control on how logic-resource hungry the DBLIF neural primitive is in terms of the size of its adder and accumulator, and by extension we can flexibly balance network complexity and cumulative resource allocation to our desire. The question arises – what about ‘corner’ neurons, if its ‘reach’ (dependent on the grid’s $$r$$ value) goes beyond the edges of the network grid? The neurons it connects to would simply wrap around to the opposite side of the grid. <br/>

Given a known $$d$$ and $$r$$, the coordinates of neighbouring neurons that a given neural unit of known coordinates must connect to can be easily determined. The following is a summary and formalisation of all the notation and parameters relevant to an RLN. <br/>

$$ n = (2⋅r + 1)^d - 1 \tag{3} $$
where $$ r ≥ 0, d ≥ 1$$, <br/>
$$r$$: radius, <br/>
$$d$$: network dimension <br/>
{: .text-center}

Nueral-Unit Coordinate: <br/> 
$$(x_1, x_2, x_3, ..., x_d) \tag{4}$$
{: .text-center}

coordinates for connecting neurons: <br/>
$$ \forall (x_1 + \lambda_1, x_2 + \lambda_2, x_3 + \lambda_3, ..., x_d + \lambda_d) | \{ \lambda_1, \lambda_2, \lambda_3, ...,\lambda_d \} \in \pm \boldsymbol{r^d} \tag{5} $$
{: .text-center}

Statement(5) basically says that the co-ordinates of the set of neighbouring neural units that a particular neural unit of coordinate $$(x_1, x_2, x_3, ..., x_d)$$ is connected to, is given by $$(x_1 + \lambda_1, x_2 + \lambda_2, x_3 + \lambda_3, ..., x_d + \lambda_d)$$ for every permutation of the set $$\{ \lambda_1, \lambda_2, \lambda_3, ...,\lambda_d \}$$, such that each $$\lambda$$ can be any value in the set 
$$\{ \pm 1, \pm 2, \pm 3, ..., \pm 𝑟 \}$$. It should be noted that each coordinate $$x$$ value should ‘wrap-around’ overflow if 
when added to a certain $$\lambda$$ it exceeds $$ \pm 𝑟$$. <br/>

Next, the issue of propagation and therefore master clock frequencies come into question. With this network architecture, a ‘lone spike’ could technically follow a recursively infinite path from an input neuron without ever reaching an output neuron. The way the DBLIF functions however, requires that a master clock cycle pass for every increment to its accumulator – a property attributed to the necessary clock driven register-memory nature of the accumulator. This means that for a theoretical ‘lone spike’ to traverse through $$n$$ number of neurons, it would take $$n$$ number of clock cycles assuming that each of said neuron’s accumulators were one clock cycle away from reaching threshold $$T$$ and firing. Thus, the necessary master clock period is not limited by the number of neurons in the network, rather the propagation delay through the ripple carry adder i.e. each neuron’s weights adder, whose size is dependent solely on $$r$$ as previously described, as well as the chosen bit-size of the weights. What is interesting to note however, is that due to the accumulator’s 1-clock cycle delay, the ‘reaction time’ of an output of an RNL to an input spike would be related to the number of neurons between the input and output. With a fast-enough master clock however this delay should be negligible, depending on the application. <br/>

In terms of applying an RLN in an FPGA, it should be noted from equation(3) that $$n$$ scales linearly with $$r$$, but exponentially with $$d$$. Not considering resource costs, the ability to vary $$d$$ allows us to insert a flexible amount of concurrent input data classes of varying dimensionality into a network. For example, consider spikes produced by the $$(r, g, b, a)$$ channels of each pixel in an input image. Each channel of each image (or ‘frame’) is a 2-dimensional data set, that we would then want to propagate through a (2+1)-dimensional network. We have 4 channels – $$r$$, $$g$$, $$b$$, $$a$$ – for each image. All 4 of these concurrent 2-dimensional channels could then be propagated through a $$d = 3 ⋅ 4 = 12$$ RLN cluster. If we wanted to process two images concurrently (for example the frame-by-frame front and back video feeds of a car simultaneously), each with their own $$(r, g, b, a)$$ channels, we would use a $$d = 24$$ RLN cluster. In each of these cases, if we assume our constituent $$d = 3$$ grids to be of side length $$= s$$, the initial input neural-layer for each channel of each image would be chosen to be any single 2-D plane of $$s \times s$$ in the grid around the centre of the multidimensional RLN cluster, ideally all close to each other. This is assuming each image is of size $$𝑠 \times 𝑠$$. The output neural-layer would then be any set of neurons near the outer perimeter of the multidimensional RLN cluster. The size and shape of said set of output neurons would depend on the application and what information the network is trying to output to an external user or system. For example, if the goal is to only determine if the car should swerve left or right (i.e. only two output classes) depending on two input video feeds each constituting of 4 channels $$(r, g, b, a)$$, then the output would only be two neurons. But if the goal is to have more complex output classification, then the arrangement of the RLN’s outer-perimeter neurons being observed as output neurons would grow in complexity as well. <br/>

A concern with RLNs, as described so far, is its potential capacity to be very noisy as a result of the widespread presence of infinitely recursive spike paths throughout a cluster of any given $$r$$ or $$d$$. Whether this widespread recursiveness is advantageous or not (say in terms of an RLN’s ability to grow implicit memory of temporally significant events), is left to be determined. But the glaring potential for noisiness remains. A potential simple remedy for this may be to link neurons to other neurons at the perimeter of its radius, as opposed to every neuron within its radius as has been discussed so far. Whether this is a viable or even helpful solution or not is also left to be determined. <br/>

Interestingly, a similar idea is touched upon in [a paper called "Multi-Dimensional Recurrent Neural Networks" by Graves, Fernandez, and Schmidhuber](https://www.researchgate.net/publication/249032122_Multidimensional_Recurrent_Neural_Networks), except with an RNN (Recurrent Neural Network) architecture called LSTM (long short term memory). [Here](https://towardsdatascience.com/only-numpy-deriving-forward-feed-on-multi-dimensional-recurrent-neural-networks-spatial-lstm-by-35d111906258) is a really nice article that also touches on Multidimensional RNN's.

</div>

