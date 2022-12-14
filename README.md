# **Generative Adversarial Networks (GANs)**

This repo contains implementations of Vanilla GAN, conditional GAN, and DCGAN using **PyTorch**.

* **Vanilla GAN** is my implementation of the original GAN paper `(Goodfellow et al.)` with certain modifications mostly in the model architecture, like the usage of LeakyReLU and 1D batch normalization.

* **Conditional GAN (cGAN)** is my implementation of the cGAN paper `(Mehdi et al.)`.
It basically just adds conditioning vectors (one hot encoding of digit labels) to the vanilla GAN.

* **DCGAN** is my implementation of the DCGAN paper `(Radford et al.)`.
The main contribution of the paper was that they were the first who made CNNs successfully work in the GAN setup.
Batch normalization was invented in the meanwhile and that's what got CNNs to work basically.


### **Generative modeling**

  1. A single variable may have a known data distribution, such as a Gaussian distribution, or bell shape. A generative model may be able to sufficiently summarize this data distribution, and then be used to generate new variables that plausibly fit into the distribution of the input variable. In fact, a really good generative model may be able to generate new examples that are not just plausible, but indistinguishable from real examples from the problem domain.
  2. `Naive Bayes` is an example of a generative model that is more often used as a discriminative model. Naive Bayes works by summarizing the probability distribution of each input variable and the output class. When a prediction is made, the probability for each possible outcome is calculated for each variable, the independent probabilities are combined, and the most likely outcome is predicted. Used in reverse, the probability distributions for each variable can be sampled to generate new plausible (independent) feature values.
  3. Other examples of generative models include `Latent Dirichlet Allocation`, or `LDA`, and the `Gaussian Mixture Model`, or `GMM`.
  4. Deep learning methods can be used as generative models. Two popular examples include the `Restricted Boltzmann Machine`, or `RBM`, and the `Deep Belief Network`, or `DBN`.
  5. Two modern examples of deep learning generative modeling algorithms include the `Variational Autoencoder`, or `VAE`, and the `Generative Adversarial Network`, or `GAN`.

Generative Adversarial Networks, or GANs, are a deep-learning-based generative model.

A standardized approach called *Deep Convolutional Generative Adversarial Networks*, or **DCGAN**, that led to more stable models was later formalized by `Alec Radford, et al.` in the 2015 paper titled “`Unsupervised Representation Learning with Deep Convolutional Generative Adversarial Networks`“.

The GAN model architecture involves two sub-models: 
1. a generator model for generating new examples and; 
2. a discriminator model for classifying whether generated examples are real, from the domain, or fake, generated by the generator model.

### **The Generator Model**
The generator model takes a fixed-length random vector as input and generates a sample in the domain.

The vector is drawn randomly from a Gaussian distribution, and the vector is used to seed the generative process. After training, points in this multidimensional vector space will correspond to points in the problem domain, forming a compressed representation of the data distribution.

This vector space is referred to as a *latent space*, or a vector space comprised of latent variables. Latent variables, or hidden variables, are those variables that are important for a domain but are not directly observable.

We often refer to latent variables, or a latent space, as a projection or compression of a data distribution. That is, a latent space provides a compression or high-level concepts of the observed raw data such as the input data distribution. 

In the case of GANs, the generator model applies meaning to points in a chosen latent space, such that new points drawn from the latent space can be provided to the generator model as input and used to generate new and different output examples.

After training, the generator model is kept and used to generate new samples.


`Random Input Vector ———> Generator model ———> Generated Example`

### **The Discriminator Model**
The discriminator model takes an example from the domain as input (real or generated) and predicts a binary class label of real or fake (generated).

The real example comes from the training dataset. The generated examples are output by the generator model.

The discriminator is a normal (and well understood) classification model.

After the training process, the discriminator model is discarded as we are interested in the generator.

Sometimes, the generator can be repurposed as it has learned to effectively extract features from examples in the problem domain. 

Some or all of the feature extraction layers can be used in transfer learning applications using the same or similar input data.

`Input Example ———> Discriminator Model ———> Binary Classification Real/Fake`

* The two models, the generator and discriminator, are trained together. The generator generates a batch of samples, and these, along with real examples from the domain, are provided to the discriminator and classified as real or fake. The discriminator is then updated to get better at discriminating real and fake samples in the next round, and importantly, the generator is updated based on how well, or not, the generated samples fooled the discriminator.

“`We can think of the generator as being like a counterfeiter, trying to make fake money, and the discriminator as being like police, trying to allow legitimate money and catch counterfeit money. To succeed in this game, the counterfeiter must learn to make money that is indistinguishable from genuine money, and the generator network must learn to create samples that are drawn from the same distribution as the training data.`”
— NIPS 2016 Tutorial: Generative Adversarial Networks, 2016.

* In this way, the two models are competing against each other, they are adversarial in the game theory sense, and are playing a zero-sum game. In this case, zero-sum means that when the discriminator successfully identifies real and fake samples, it is rewarded or no change is needed to the model parameters, whereas the generator is penalized with large updates to model parameters.

* Alternately, when the generator fools the discriminator, it is rewarded, or no change is needed to the model parameters, but the discriminator is penalized and its model parameters are updated.
At a limit, the generator generates perfect replicas from the input domain every time, and the discriminator cannot tell the difference and predicts “unsure” (e.g. 50% for real and fake) in every case. This is just an example of an idealized case; we do not need to get to this point to arrive at a useful generator model.


### **GANs and Convolutional Neural Networks**
* GANs typically work with image data and use *Convolutional Neural Networks*, or CNNs, as the generator and discriminator models.
Modeling image data means that the latent space, the input to the generator, provides a compressed representation of the set of images or photographs used to train the model. It also means that the generator generates new images or photographs, providing an output that can be easily viewed and assessed by developers or users of the model.

### Deep Convolutional Generative Adversarial Network - **DCGAN**
Literally nothing changed in the training loop of DCGAN compared to vanilla GAN:

    Things that changed:
        * Model architecture - using CNNs compared to fully connected networks
        * We're now using CelebA dataset loaded via utils.get_celeba_data_loader (MNIST would work, it's just too easy)
        * Logging parameters and number of epochs (as we have bigger images)

### DCGAN implementation
    
    Note1:
        Many implementations out there, including PyTorch's official, did certain deviations from the original arch,
        without clearly explaining why they did it. PyTorch for intance uses 512 channels initially instead of 1024.
    
    Note2:
        Small modification I did compared to the original paper -- I used kernel size = 4 as I can't get 64x64
        output spatial dimension with 5 no matter the padding setting. I noticed others did the same thing.
        Also I'm not doing 0-centered normal weight initialization -- it actually gives far worse results.
        Batch normalization, in general, reduced the need for smart initialization but it obviously still matters.
        
        
### **Conditional GANs**
An important extension to the GAN is in their use for conditionally generating an output.

The generative model can be trained to generate new examples from the input domain, where the input, the random vector from the latent space, is provided with (conditioned by) some additional input.
The additional input could be a class value, such as male or female in the generation of photographs of people, or a digit, in the case of generating images of handwritten digits.
In this way, a conditional GAN can be used to generate examples from a domain of a given type.

Taken one step further, the GAN models can be conditioned on an example from the domain, such as an image. This allows for applications of GANs such as text-to-image translation, or image-to-image translation. This allows for some of the more impressive applications of GANs, such as style transfer, photo colorization, transforming photos from summer to winter or day to night, and so on.

### **Why Generative Adversarial Networks?**
One of the many major advancements in the use of deep learning methods in domains such as computer vision is a technique called **data augmentation**.

Data augmentation results in better performing models, both increasing model skill and providing a regularizing effect, reducing generalization error. It works by creating new, artificial but plausible examples fro
m the input problem domain on which the model is trained.

In complex domains or domains with a limited amount of data, generative modeling provides a path towards more training for modeling. GANs have seen much success in this use case in domains such as deep reinforcement learning.

GANs’ successful ability to model high-dimensional data, handle missing data, and the capacity of GANs to provide multi-modal outputs or multiple plausible answers.

Perhaps the most compelling application of GANs is in conditional GANs for tasks that require the generation of new examples. Here, *Goodfellow* indicates three main examples:

* **Image Super-Resolution**. The ability to generate high-resolution versions of input images.

* **Creating Art**. The ability to create new and artistic images, sketches, painting, and more.

* **Image-to-Image Translation**. The ability to translate photographs across domains, such as day to night, summer to winter, and more.


### **Summary**
A Generative Adversarial Network, or GAN, is a type of neural network architecture for generative modeling.

Generative modeling involves using a model to generate new examples that plausibly come from an existing distribution of samples, such as generating new photographs that are similar but specifically different from a dataset of existing photographs.

A GAN is a generative model that is trained using two neural network models. One model is called the “*generator*” or “generative network” model that learns to generate new plausible samples. The other model is called the “*discriminator*” or “discriminative network” and learns to differentiate generated examples from real examples.

The two models are set up in a contest or a game (in a game theory sense) where the generator model seeks to fool the discriminator model, and the discriminator is provided with both examples of real and generated samples.

After training, the generative model can then be used to create new plausible samples on demand.
