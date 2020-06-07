---
layout: post
title:  "MVP Pattern explained with a Kotlin Android App (Part 1)!"
date:   2018-04-22 21:39:49 +0100
categories: kotlin
---


Introduction

Over the years, i have seen many implementations the MVP pattern in android projects. Here, i will introduce a pattern i have found to be useful in my projects.

It is a highly opinionated implementation that depends on the use of certain libraries in android like Dagger2.

What is MVP and Why should i care.

MVP(Model View Presenter) is an architectural pattern that coordinates interactions between Models and Views via a Presenter.

Android Views are tightly coupled with the data they display. Recyclerviews, viewpagers etc. expect adapters. Redesigning your UI or simply moving things around can require complete refactor of code everytime as a new View might have it own way of accepting and accessing data.

Because of this, it is important to have a layer where you can manipulate your data and another where your layout the views and user interaction. This is what MVP provides.

What we will be building.

In this project, we will be building a bible app in Kotlin. The code is available on Github.

In the next post, i will layout the app plan and explain how the app is implemented with the MVP pattern.