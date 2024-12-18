# GAN-to-POWDER
This code is to generate the results in the technical report at https://wp.ece.uw.edu/wp-content/uploads/sites/36/2024/11/Techical-Report-GAN-to-POWDER.pdf


## Generator Model Parameters

- Generator Model saved at 1 iteration : https://drive.google.com/file/d/1gQqTiDXsdC3cjQ8spv885-T4leYbLY4j/view?usp=share_link
- Generator Model saved at 60000 iterations : https://drive.google.com/file/d/1UpOyWCd6g_yglTnZ50Qm2wcAXNmEHhvU/view?usp=share_link
 
We have uploaded the drive link which will direct to the model parameter files, because the Generator model paramter file's size is greater than 25 MB,
for which Github doesn't allow to upload file size greater than 25 MB.


## POWDER data description

The datasets are stored in HDF5 files containing both metadata and raw IQ samples for
each link at which data was collected. The client transmit to BS with the following frame schedule: `GGGGGGGPGGGGGGG`

which includes $15$ slots.  $14$ out of $15$ slots are guard intervals (G letters) while one slot includes a pilot signal (P letter). 
The pilot signal includes $64$ LTS pilot symbols. Each LTS symbol includes $64$ samples. The BS receives the pilot slot from both of its antennas. 
Each dataset containing $2$ sub-datasets for $2$ links has the following dimensions: $(4000, 1, 1, 2, 8192)$. 
$4000$ is the number of frames collected in each dataset, 
$1$ is the number of BSs, 
$1$ is the number of client antennas, 
$2$ is the number of BS antennas, 
and 8192 is the number of I and Q samples in the received samples in each frame.

Thus:

- Each frame contains $64$ repetitions of LTS.
- FFT size = $64$ $\rightarrow$ Each LTS uses $64$ subcarriers;
- Total \# of IQ samples per frame = $64$ LTS per frame $\times$ $64$ subcarriers per LTS = $4096$ IQ samples per frame;
- The dimension of the provided dataset = $8192 \times 2 \times 1 \times 1 \times 4000$, where $8192 = 2 * 4096$ (2 * indicates I and Q components, and $4096$ is total \# of IQ samples per frame), $2$ is \# of BS antennas, $1$ is \# of BSs, $1$ is \# of UEs. $4000$ is \# of frames.
    
We select a POWDER dataset for the link - Location $8$ to the $1^{st}$ antenna of BS "EBC" as the real dataset used for GAN training}, 
where Location $8$ and BS "EBC" are shown below:


![Alt Text](https://github.com/liucaouw/GAN-to-POWDER/blob/main/POWDER_map.png)


Note that there are a total of $14 \times 5 \times 2 = 140$ links ($14$ client locations, $5$ BS locations, and $2$ antennas per BS) for the collected dataset. 
We need to train $140$ separate GANs for the $140$ collected datasets. The proposed GAN algorithm can be applied to all $140$ datasets. 
Here, we train the GAN and evaluate its performance based on one specific link dataset of file : `trace-2020-9-23-13-34-41_1x2x1_LOC8.hdf5`


github.py is the file which can be used to train our GAN model on the POWDER dataset hdf5 file.

Looking at GAN_to_POWDER_code.ipynb,

we need to read the hdf5 file to know about how the POWDER data files are stored.

Once we read the hdf5 file and get the IQ samples data as a matrix of $\mathbb{R}^{4000 \times 8192}$, 
where each row of the matrix is vector of $1$ LTS frames containing $4096$ IQ samples of the form:
 
 $$ \[I_1, Q_1, I_2, Q_2, \ldots I_{4096}, Q_{4096}\]_{1 \times 8192}$$


Now, for the GAN model we have the following architecture for Generator and Disrcriminator

## Generator

![Alt Text](https://github.com/liucaouw/GAN-to-POWDER/blob/main/G_A_T.png)


## Discriminator

 ![Alt Text](https://github.com/liucaouw/GAN-to-POWDER/blob/main/D_A_T.png)

## Training parameters

 - Input Noise dimension for Generator input: $200$
 - Loss function: Binary Cross Entropy Loss
 - Optimizer: ADAM
 - Learning rate: $0.0002$
 - $\beta_1$ ($1^{st}$ order Moment Decay in ADAM): $0.5$
 - $\beta_2$ ($2^{nd}$ order Moment Decay in ADAM): $0.999$ (Tensorflow default)
 - Batch size = 512 (256 Real LTS vectors + 256 Generated LTS vectors)

## Saving and Loading Models

Looking at the **Training code** section in the .py file, we are training the model for $60000$ iterations 
and saving our model at every $10000$ iterations. We are doing this because if the Google Colab Runtime 
sessions crash or stop, we can load the model at the multiple of $10000$ iterations (say at $20000$ iterations if the runtime crashed), 
we can load the model saved at $20000$ iterations and resume our training till $60000$ iterations 
(in this case we need to run only $40000$ iterations from the loaded checkpoint).

So, we have included the section **Loading saved models** in the .py, 
where if we provide the path where the saved Generator and Discriminator models are stored, 
we can resume trainng the from the loaded checkpoint, when we re-run the **Training code** section again, 
but we need to adjust the **iterations** variable in the **Training code** for our training to run for 
$60000$ iterations.


## Plot the Generator and Discriminator Loss

Once we run the trained the GAN model, we can see the Loss curves of the Generator and 
the Discriminator for the iterations we ran with respect to the loaded checkpoint we
started.

## Generating vectors for required number and recording the time taken to do so

After training, or loading the trained GAN model parameters, the code in this section 
produces Generated LTS vectors of our size/quantity we want, as mentioned in the **gen_data_size** variable. This code 
also provides you the time taken to generate such vectors in Google Colab output cell.

## Function to separate corresponding I and Q samples from the LTS frames represented as vectors

We generate the data in the form, 

$$
\begin{bmatrix}
I_{1, 1}, & Q_{1, 1}, & I_{1, 2}, & Q_{1, 2}, & \cdots & I_{1, 4096}, & Q_{1, 4096} \\
I_{2, 1}, & Q_{2, 1}, & I_{2, 2}, & Q_{2, 2}, & \cdots & I_{2, 4096}, & Q_{2, 4096} \\
\vdots & \vdots & \vdots & \vdots & \ddots &  \vdots & \vdots \\
I_{n, 1}, & Q_{n, 1}, & I_{n, 2}, & Q_{n, 2}, & \cdots & I_{n, 4096}, & Q_{n, 4096}
\end{bmatrix}_{n \times 8192}
$$

$n$ being the \# of vectors we want to generate.


The following function outputs I and Q matrices of the form :

$$
I = \begin{bmatrix}
I_{1, 1}, & I_{1, 2}, & \cdots & I_{1, 4096} \\
I_{2, 1}, & I_{2, 2}, & \cdots & I_{2, 4096} \\
\vdots & \vdots & \ddots & \vdots \\
I_{n, 1}, & I_{n, 2}, & \cdots & I_{n, 4096}
\end{bmatrix}_{n \times 4096}
$$

$$
Q = \begin{bmatrix}
Q_{1, 1}, & Q_{1, 2}, & \cdots & Q_{1, 4096} \\
Q_{2, 1}, & Q_{2, 2}, & \cdots & Q_{2, 4096} \\
\vdots & \vdots & \ddots & \vdots \\
Q_{n, 1}, & Q_{n, 2}, & \cdots & Q_{n, 4096}
\end{bmatrix}_{n \times 4096}
$$

This is done for easier calculation or interpretation when we calculate the Average Power of the Vectors 
and Variance of the Power of the Vectors.


## Average Power of the Generated vectors

This code gives you the Average Power of the LTS frame vectors, by running the function in 
section **Function to calculate Average Power of the LTS frames represented as vectors**

## Variance of the Power of the Generated vectors

This code gives you the Variance of the Power of the LTS frame vectors, by running the function in 
section **Function to calculate the Variance of the Power of the LTS frames represented as vectors**



 

