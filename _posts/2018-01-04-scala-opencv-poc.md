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

### Project setup with SBT

Simply add plugin in `project/plugins.sbt`

```scala
addSbtPlugin("org.bytedeco" % "sbt-javacv" % "1.15")
```