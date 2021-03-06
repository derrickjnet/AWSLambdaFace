AWSLambdaFace: Serverless Face Recognition
---

### Goal:

Build an elastic (i.e. horizontally scalable) system that uses deep
convolutional neural networks to perform face detection and recognition in the
cloud.

### Description:

Serverless compute platforms such as Amazon Web Services (AWS) lambda were
intended to be used for web microservices and to asynchronously handle events
generated by AWS services (S3, DynamoDB, etc.). However, AWS lambda allows users
to upload arbitrary linux binaries along with their lambda functions. These
binaries can be executed during a lambda invocation, effectively turning AWS
lambda into a supercomputer that can be started on-demand in seconds and billed
at a 100ms granularity.

We take advantage of this *feature* and run a full-blown deep convolutional
neural network based face recognition tool
([openface](https://cmusatyalab.github.io/openface/)) on AWS lambda. The code in
this repository gives you everything you need to deploy this system onto your
AWS account and start detecting/recognizing faces in jpeg images! (with zero
configuration or model tuning!)

### The bigger vision:

In the AWS region `us-east-1`, a normal user can run up to 600 lambda functions
concurrently (as of April 10th, 2017). This enables our system to perform 600
face recognition operations at the same time! (and more parallelism is available
if the system is deployed to additional AWS regions!) The ability to scale out
horizontally with virtually no additional effort is a tremendous asset; for
example, we can perform face recognition across every frame in a video or
provide a scalable face recognition microservice as part of a larger
application. The possibilities are endless!
<!-- (and check out [ExCamera](http://ex.camera) if you want a glimpse of some
of these possibilities!) -->

### Prerequisites:

1. You must **have an AWS account** to run this code. Create an account [here](http://aws.amazon.com) if you do not have one already.
2. You must also **create an AWS access key** to use with awscli (a commmandline tool for managing your AWS account). To do this follow the instructions on [this page](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html).

### Setup:

```bash
# install pip and virtualenv if necessary
$ curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
$ python get-pip.py
$ pip install virtualenv

# clone the repo and install awscli and boto3
$ git clone https://github.com/excamera/AWSLambdaFace.git
$ cd AWSLambdaFace
$ virtualenv .venv
$ source .venv/bin/activate
$ pip install -r requirements.txt # installs awscli and boto3

# setup the awscli (create an access key and enter details below)
$ aws configure
AWS Access Key ID []: ****************ANWQ
AWS Secret Access Key []: ****************JawS
Default region name []: us-east-1
Default output format [None]:

# download the binary blobs necessary for the face recognizer
$ ./blobs/get_blobs.sh
downloading... (lots of output)

# deploy the face recognition lambdas to your account
$ ./install_lambdas.sh
creating bucket
uploading dependencies to s3
upload: blobs/deps.zip to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/deps.zip
upload: blobs/root-495M-2017-02-06.tar.gz to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/root-495M-2017-02-06.tar.gz
upload: blobs/lfw_face_vectors.csv.gz to s3://9a480eb7-be5b-4e15-81fe-4de41323907e/lfw_face_vectors.csv.gz
adding 'lambda-executor' role
adding 'prepare-face-recognizer' lambda
adding 'recognize-face' lambda
Done!
```

```bash
# remove the deployed lambdas when you are done
$ ./uninstall_lambdas.sh
removing s3 bucket
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/lfw_face_vectors.csv.gz
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/deps.zip
delete: s3://9a480eb7-be5b-4e15-81fe-4de41323907e/root-495M-2017-02-06.tar.gz
removing 'lambda-executor' role
removing 'prepare-face-recognizer' lambda
removing 'recognize-face' lambda
Done!
```

### Usage (code located in `scripts` directory):

```bash
# generating a model for a face
$ ./scripts/train_face_recognizer.py --help
usage: ./train_face_recognizer.py IMAGE.jpg > FACEVECTORS.csv
description:
    returns the augmented feature vectors for the face in IMAGE.csv.
    Assume exactly one face in the image.

# using a model to recognize a face in an image
$ ./scripts/recognize_face.py --help
usage: ./recognize_face.py FACEVECTORS.csv IMAGE.jpg
description:
    returns `True` or `False` if the face used to generate
    FACEVECTORS.csv is present in IMAGE.csv.
```

### Simple example (code located in `scripts` directory):

To see how things work let's jump into a simple example. The following snippets
will generate a model to recognize [John Emmons'](http://johnemmons.com) face,
then use the model to check if John's face is in a photo.

#### Model generation

Training image              |
:---------------------------:
![](README.pics/john0.jpg)  |
`scripts/pics/john0.jpg`    |

```bash
# generate a model for John's face
$ cd scripts
$ ./train_face_recognizer.py pics/john0.jpg > john.model.csv
reading input file
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker

Done!
```

#### Recognition

Training image              | Test image                 | Faces match?
:--------------------------:|:--------------------------:|:----------------------------:
![](README.pics/john0.jpg)  | ![](README.pics/john1.jpg) | ![](README.pics/correct.jpg)
`scripts/pics/john0.jpg`    | `scripts/pics/john1.jpg`   | 

```bash
# use the model to see if John's face is present in an image
$ ./recognize_face.py john.model.csv pics/john1.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker

Faces match: True
```

Training image              | Test image                 | Faces match?
:--------------------------:|:--------------------------:|:----------------------------:
![](README.pics/john0.jpg)  | ![](README.pics/john2.jpg) | ![](README.pics/correct.jpg)
`scripts/pics/john0.jpg`    | `scripts/pics/john2.jpg`   | 

```bash
$ ./recognize_face.py john.model.csv pics/john2.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker

Faces match: True
```
Training image              | Test image                     | Faces match?
:--------------------------:|:------------------------------:|:----------------------------: 
![](README.pics/john0.jpg) | ![](README.pics/jim-carrey.jpg) | ![](README.pics/incorrect.jpg)
`scripts/pics/john0.jpg`    | `scripts/pics/jim-carrey.jpg`  | 

```bash
$ ./recognize_face.py john.model.csv pics/jim-carrey.jpg
reading input files
connecting to AWS lambda
waiting for remote lambda worker to finish
reading result from remote lambda worker

Faces match: False
```

### About Me:

Hi! My name is [John Emmons](http://johnemmons.com); I am a computer science PhD
student at Stanford University who works on problems at the intersection of
computer system and machine learning (primarily computer vision).

If you like this work and want to learn more about it, feel free to drop me a
line a [mail@johnemmons.com](mailto:mail@johnemmons.com). And if this work
inspires you to do your own related work, please cite our research group! (see
below)

### Acknowledgements:

This work was done as part of preparing a demonstration for ExCamera
[[pdf](https://www.usenix.org/system/files/conference/nsdi17/nsdi17-fouladi.pdf)],
which was presented at [NSDI'17](https://www.usenix.org/conference/nsdi17). I
would like to thank all of the authors on the ExCamera paper for giving me
inspiration for this project and for helping me with the system design aspects.

I would also like to give special thanks to [Sadjad Fouladi](http://sadjad.org/)
and [Deepak Narayanan](http://cs.stanford.edu/~deepakn/) for their invaluable
comments while I wrote this README. And finally I want to give a shout-out to
[Luiz Gustavo de Souza](https://github.com/luicaps) for pointing out grammatical
and spelling mistakes in the README.

### Citation:

```
@inproceedings {201559,
  author = {Sadjad Fouladi and Riad S. Wahby and Brennan Shacklett and Karthikeyan Vasuki Balasubramaniam and William Zeng and Rahul Bhalerao and Anirudh Sivaraman and George Porter and Keith Winstein},
  title = {Encoding, Fast and Slow: Low-Latency Video Processing Using Thousands of Tiny Threads},
  booktitle = {14th USENIX Symposium on Networked Systems Design and Implementation (NSDI 17)},
  year = {2017},
  isbn = {978-1-931971-37-9},
  address = {Boston, MA},
  pages = {363--376},
  url = {https://www.usenix.org/conference/nsdi17/technical-sessions/presentation/fouladi},
  publisher = {USENIX Association},
 }
 ```