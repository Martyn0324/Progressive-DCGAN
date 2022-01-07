# Cocogoat - A Progressive DCGAN
Progressive DCGAN created on Pytorch, based on many different codes. The only originality here is the name and the DatasetCreator class and the prototype file...perhaps the AI to label the dataset, too, but I'm pretty sure that, though I didn't see anyone using this tactic to label a big dataset, I'm not the first one using it.

The name Cocogoat it's because I've been testing this code initially using a dataset of 10.000 images of Ganyu from Genshin Impact.

*OBS: If you read the reference papers and check this code, you'll notice that some resources are missing. The explanation for this is...simply because I don't know how to do it. That's specially the case for layer fading from NVidia's paper.*

## How it works

The file data_selector.py consists of the creation of the dataset(originally 100x100 Ganyu fanarts downloaded using [gallery-dl](https://github.com/mikf/gallery-dl)) and the creation of a multi-class neural network to classify images according to its quality and if they're not Ganyu fanarts. 1000 images are passed to a train dataset and manually labeled accordingly.

After training for a small time(the performance stabilizes at around 500), the neural network is used to label the remaining 9000 images. All undesired images can then be eliminated. The remaining ones are then resized so they can be passed into the ProGAN to generate images.

The ProGAN, in a nutshell, begins to train with low resolution images(4x4), and then progressively trains on higher resolution images(8x8 for level 1, 16x16 for level 2, 32x32 level 3 and so forth). This strategy makes it easier for the Neural Network to learn patterns as the weights are initially adjusted with simple data and only when the weights are properly calibrated they are adjusted with more complex data.


## Prototype

I've never seen any image generating GAN using LSTMs, so I decided to give it a chance. Perhaps LSTMs aren't used for images simply because it isn't worth it, but I'm still eager to give it a try and see it for myself.

Instead of passing sequences of data directly into the LSTM layer, I've decided to let the GAN try to generate and image from noise using the transposed 2D convs, and, at the end of the neural network, when the image has already been generated, pass it into a LSTM layer. The idea here is to make the Neural Network try to predict correctly what is the best pixel value to be added into an image based on what has been created so far.

Suppose that the final tranposed conv 2D layer returned certain image. We'll have an image with 3 channels(RGB) and height x width dimensions.
Each channel would be something like this:

![rubbish](https://user-images.githubusercontent.com/28028007/148013549-2ae06096-b728-4647-b757-0e4c4a9d5ac8.png)

That is, a grid, which could also be seen as a table, just like a DataFrame or Series when we work with Pandas, where we got X (height) and Y labels(width). Each value of X is assigned to a value of Y. That can be more easily visualized if you think about price prediction, for example. You got a table of values, with X being the open price of that asset that day, and Y being the close price. You want to use the open price to predict the close price. Then, you have a table with X values and Y values, convert them into a sequence dataset, pass into a LSTM and voilá.
With images, I'm simply considering that I have the same table, but, in a 8x8 image, I'll have a table with 8 columns and 8 values. Some of those columns can be considered my X to predict the Y, the remaining ones.

Considering that, we could pass an image like this into a LSTM

![illustration](https://user-images.githubusercontent.com/28028007/148013037-664707cf-75b9-45ca-8bd5-618d84139760.png)

So it could predict what is missing based on what it already has:

![scheme](https://user-images.githubusercontent.com/28028007/148014309-9f8b4bf2-864b-4238-a2fa-fb2a12905d63.png)

Unfortunately, though, I was stupid and didn't consider that creating sequences would make me get more data than an array (1,3,8,8) could support, so I have to pass the output of that LSTM to another tranposed conv 2D, in order to filter some data and if everything goes ok, I could get something like this:

![complete](https://user-images.githubusercontent.com/28028007/148014411-2ea06314-7300-4937-a63b-ad81d6b48a5d.png)

As the name suggests, this is just a prototype. Perhaps it would be more profitable to, instead of inserting the LSTM into the Generator, to create a second Generator with those LSTMs. I'll see how this goes with time.

*None of those images have been generated by this neural network. I'm just trying to explain my idea. Maybe someone could think about something better based on that*

That being said, I'm currently testing this hypothesis...and I'm not a researcher in this area, which means I might not dedicate that much time into this...


## References:
**Nathan Inkawhich. Pytorch's DCGAN Tutorial:** https://pytorch.org/tutorials/beginner/dcgan_faces_tutorial.html

**Tim Salimans, Ian Goodfellow, Wojciech Zaremba, Vicki Cheung, Alec Radford and Xi Chen. Improved Techniques for Training GANs:** https://arxiv.org/pdf/1606.03498.pdf

**Tero Karras, Timo Aila, Samuli Laine and Jaakko Lehtinen. PROGRESSIVE GROWING OF GANS FOR IMPROVED QUALITY, STABILITY, AND VARIATION:** https://arxiv.org/pdf/1710.10196.pdf

*Some classes from Didática Tech(PT-BR) about DCGAN: https://didatica.tech/*

**Florian Dedov(AKA Neural Nine):** https://www.youtube.com/watch?v=GFSiL6zEZF0 (Thanks for finally making me understand how these annoying, nitpicking LSTM layers work)
