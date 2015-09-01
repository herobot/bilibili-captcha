# bilibili-captcha

## Overview

The Python program attempts to recognize [CAPTCHA images](http://www.bilibili.com/captcha) generated by [bilibili.com](http://www.bilibili.com/).

The program is developed and tested on `Mac OS X 10.10.4` and `Windows 10` using `Python 3.4` interpreter. There might be problems if the program is run under other environments. Particularly, the program will not compile using `Python 2` because of the differences in syntax.

## Dependencies

- [matplotlib](http://matplotlib.org)
- [numpy](http://www.numpy.org)
- [pillow](https://python-pillow.github.io/)
- [requests](http://www.python-requests.org/en/latest)
- [scikit-learn](http://scikit-learn.org/stable)
- [scipy](http://www.scipy.org)
- [theano](http://deeplearning.net/software/theano)

## Usage

Clone all the files and put them under the same directory as your program, including the `dataset` folder.

## Key Files Overview

**`captcha_provider.py`**

This module represents the source of the CAPTCHA. An abstract base class `HttpCaptchaProvider` is defined to represent abstract CAPTCHA providers and a derived class `BilibiliCaptchaProvider` is defined to represent the specific CAPTCHA source one is attempting to recognize. If you want to define another CAPTCHA source, you can define another class that inherits the abstract base class. Two methods that must be overridden are `_get_data_from_seq` and `_is_correct_response`. `_get_data_from_seq` extracts the necessary data that need to be submitted as part of the request to verify. `_is_correct_response` checks whether the recognized character sequence is correct.

**`captcha_recognizer.py`**

This module is designed to recognize the images (the return value of `imread` function in `matplotlib.image`) from a particular CAPTCHA source (derived class of `HttpCaptchaProvider`). The steps to recognize an image are

1. Noise reduction

  1.1 Noise reduction by hsv (`remove_noise_with_hsv` function)

  1.2 Noise reduction by neighbors (`remove_noise_with_neighbors` function)

  After this step the remaining image is a greyscale image with slight noise.

2. Segmentation

  Components are found using the next-nearest-neighbor. (`segment_with_label` and other commands)

  After this step the image is partitioned into a number of "characters".

3. Recognition

  If the number of partitioned characters is as desired, each is recognized using the model in `captcha_learn.py`. (`recognize` function)

**`captcha_learn.py`**

This module is designed to use a multilayer perceptron (MLP) model to learn to recognize individual CAPTCHA character.
The input layer consists of 300 neurons, which is the result of flattening the standard 20 by 15 CAPTCHA character image.
There is only one hidden layer, and it consists of 200 hidden neurons.
The output layer consists of 26 neurons, each corresponding to one possible outcome (`EFGH JKLMN PQR TUVWXY  123456 89`).
The activation function of hidden units is `tanh`.
The activation function of output units is the softmax function.
The model is trained by using stochastic gradient descent with minibatches.
There is already a model with tuned parameters, and it is saved in `best_model.pkl`.
If you want to reconstruct the model, call `reconstruct_model` method.
If you want to make prediction, call `predict` method.

## Example

Let's take a look at a few examples.

Suppose we want to recognize a bilibili CAPTCHA image called `img` (the return value of `imread` in `matplotlib.image`), we can

```python
import time

import config as c
import dataset_manager
from captcha_recognizer import CaptchaRecognizer
from captcha_provider import BilibiliCaptchaProvider
from helper import show_image, time_func
import captcha_learn

provider = BilibiliCaptchaProvider()
recognizer = CaptchaRecognizer()
success, seq, weak_confidence = recognizer.recognize(img, save_intermediate=True, verbose=True,reconstruct=False, force_partition=False)
```

For the parameters,
`save_intermediate` controls whether intermediate files should be saved.
If set to true, those files will be created under directory `/temp`.
`verbose` controls whether additional information like timing is printed out.
`reconstruct` controls whether reconstructing the learning model is needed.
`force_partition` controls whether the recognizer recognizes those images that are partitioned into four pieces.
This parameter is designed to be used in cases under which we want to maximize our chance of success when we cannot
overlook the number of times the partition fails. However, in real world situations,
it is usually the case that we can regenerate CAPTCHA as many times as we want, so we do not have to worry about the number of times the partition fails.

For the return values,
`success` is a boolean variable indicating whether the recognition is successful.
`seq` is the recognized sequence if the recognition is successful.
`weak_confidence` is a boolean variable indicating whether we are only weakly confident about the partition.
It is not relevant here because the variable force_partition is set to be false.

Now suppose we want to test whether a CAPTCHA fetched from bilibili.com is correct for n times, we can

```python
import time

import config as c
import dataset_manager
from captcha_recognizer import CaptchaRecognizer
from captcha_provider import BilibiliCaptchaProvider
from helper import show_image, time_func
import captcha_learn

test_recognize_http(num=n)
```

For an explanation of output, see section "Benchmark" below.   


## Benchmark

In a benchmark test on an Amazon EC2 instance, 1000 CAPTCHA images were fetched from bilibili.com (`test_recognize_http(num=1000)`). The recognition result is as follows:

```
Fail:  233
Right weak:  78
Right strong:  412
Right total:  490
Wrong weak:  260
Wrong strong:  17
Wrong total:  277
Total success rate:  0.49
Success rate when confident:  0.6388526727509778
Success rate when strongly confident:  0.9603729603729604
Success rate when weakly confident:  0.23076923076923078
Time used to test recognize http is:  350.77492213249207
```

According to the result, 490 out of 1000 CAPTCHAs were recognized successfully, and therefore the total success rate is about 49%. We also notice that when the model is strongly confident of the result, i.e. when the image can be successfully partitioned into 5 characters, the success rate is as high as 96%. Therefore, in practical situations where the total success rate may not be as important because normally one could ask the server to regenerate the CAPTCHA if not strongly confident, the success rate would be nearly 100%.

## Related Info

[Classifying MNIST digits using Logistic Regression](http://deeplearning.net/tutorial/logreg.html)

[Deep Learning Tutorial](http://deeplearning.net/tutorial/contents.html)

[Image manipulation and processing using Numpy and Scipy](http://scipy-lectures.github.io/advanced/image_processing/)

[Multilayer Perceptron](http://deeplearning.net/tutorial/mlp.html)

[Scikit-image: image processing](http://scipy-lectures.github.io/packages/scikit-image/)

## Authors

The authors of the software are listed in [AUTHORS](AUTHORS).

## License

This software is licensed under the MIT License. See [LICENSE](LICENSE).
