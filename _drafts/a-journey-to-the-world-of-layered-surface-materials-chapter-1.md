---
layout: post
title: A Journey to the World of Layered Surface Materials, Chapter 1
---

I decided to implement some kind of multi-layered material (BSDF) models in my pet renderer since they offer much more expressivity than basic models (e.g. smooth Fresnel, glossy micro-facet models, or diffuse Lambert). At the beginning I wanted to directly implement the paper [Arbitrarily Layered Micro-Facet Surfaces](https://www.cg.tuwien.ac.at/research/publications/2007/weidlich_2007_almfs/) by Andrea Weidlich and Alexander Wilkie; however, I found their approach too ad-hoc so I decided to come up with my own model, which is heavily inspired by the ideas from the paper.