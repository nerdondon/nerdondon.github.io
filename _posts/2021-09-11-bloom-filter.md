---
layout: post
title: '[Notes]: Bloom Filters'
description: 'Notes on Bloom filters'
tags: notes bloom filter leveldb concurrency
---

## Prelude

These are just some of my notes and references on Bloom filters while doing research for building an
LSM tree based database.

## Notes

- A Bloom filter is a probablistic data structure that allows checks for set membership. These
  checks for set membership can return false positives but will never produce false negatives. More
  clearly: "if a negative match is returned, the element is guaranteed not to be a member of the
  set" (Alex Petrov - Database Internals).

- Used to lower I/O costs incurred by the tiered structure of LSM-trees

## Useful references

- [Alex Petrov - Database Internals](https://databass.dev)
- [Jamie Talbot - What are Bloom filters?](https://blog.medium.com/what-are-bloom-filters-1ec2a50c68ff)
- [Onat - Let's Implement a Bloom Filter](https://onatm.dev/2020/08/10/let-s-implement-a-bloom-filter/)
  - [Github - plum - A Rust implementation of a Bloom filter](https://github.com/distrentic/plum)
