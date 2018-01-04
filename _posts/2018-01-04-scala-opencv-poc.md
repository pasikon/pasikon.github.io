---
title: "Image resizing and serializing with Scala & OpenCV3"
excerpt: "Simple PoC for Scala & OpenCV3 & Java API interoperability"
last_modified_at: 2018-01-03T11:20:58-05:00
header:
  teaser: "assets/images/opencv_logo.png"
tags: 
  - scala
  - opencv
  - sbt
toc: true
---

[^1]: <https://github.com/bytedeco/sbt-javacv>

Recently I needed to setup a stream of images to feed my trained CNN (Convolutional Neural Network). At the time of training the model I was using OpenCV library with Python for image manipulation. 

Now having Tensorflow model exported I wanted to integrate it with some infrastructure. 

So the first challenge is to find out how to use OpenCV with Scala, then to figure out how to serialize it properly and also check how this all refers to Java API. Since image manipulation is not something I do in Scala every day, here is some quick research :)  

[Full example](https://github.com/pasikon/opencv-scala)

### Project setup with SBT

I used `sbt-javacv`[^1]

Simply add plugin in `project/plugins.sbt`

```scala
addSbtPlugin("org.bytedeco" % "sbt-javacv" % "1.15")
```

### Reading image from file

```scala
val matSrc: Mat = opencv_imgcodecs.imread("/Users/mzurek/PycharmProjects/cnn_finetune/seedl_data/test/0c27cf05f.png")
```

`Mat` is `OpenCV` image representation as matrix.

Image is random size, `depth = 8, channels = 3`

### Resize image

```scala
val _width = 150
val _height = 150

val matDst = new Mat()

val size = new Size(_width, _height)
opencv_imgproc.resize(matSrc, matDst, size)
```

`matDst` holds image resized to `150 x 150`

### Serialization

#### Serialize

What I need to get is `Array[Bytes]` from `Mat`:

```scala
val darr: Array[Byte] = new Array[Byte]((matDst.total() * matDst.channels()).toInt)
matDst.data().get(darr)
```

#### Deserialize

Now, let's check if I have done it properly, from obtained `Array[Bytes]` I want to create another `Mat`, serialize it again and create image object with Java API. Displaying image that Java object is holding will prove correct OpenCV, Scala & Java API usage

```scala
val myPic = new Mat(size, opencv_core.CV_8UC3)
myPic.data().put(darr, 0, darr.length)

val darrMyPic: Array[Byte] = new Array[Byte]((myPic.total() * myPic.channels()).toInt)
myPic.data().get(darrMyPic)
``` 

#### As Java object & display

```scala
try {
	val bufImage = new BufferedImage(_width, _height, BufferedImage.TYPE_3BYTE_BGR)
	bufImage.getRaster.setDataElements(0, 0, _width, _height, darrMyPic)

	val frame = new JFrame
	val icon = new ImageIcon(bufImage)
	frame.getContentPane.add(new JLabel(icon))
	frame.pack()
	frame.setVisible(true)
	} catch {
		case e: Exception =>
		e.printStackTrace()
	}
}
```
