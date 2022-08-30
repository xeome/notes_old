---
title: XDP-Tutorial
tags:
  - '#linux'
toc: true
season: summer
date updated: 2022-08-30 17:14
---

Links: [[Linux]]

# XDP-Tutorial

## Introduction

XDP is an upstream Linux kernel component that allows users to install packet processing programs into the kernel. The programs are written in restricted C and compiled into eBPF byte code. Read the academic paper (pdf) or the Cilium BPF reference guide for a general introduction to XDP.

This tutorial aims to provide a hands-on introduction to the various steps required to create useful programs with the XDP system. We assume you know the basics of Linux networking and how to configure it with the iproute2 suite of tools, but you have no prior experience with eBPF or XDP. All of the lessons are written in C, and they cover basic pointer arithmetic and aliasing. This tutorial is intended to be a hands-on introduction to the various steps required to successfully write useful programs using the XDP system.

Please keep in mind that this tutorial was written by a university first-year computer science student who has only recently begun learning XDP.

## Dependencies

For basic dependencies refer to <https://github.com/xdp-project/xdp-tutorial/blob/master/setup_dependencies.org>.

You will also need xdp-tools. If your distribution repositories lack xdp-tools, you can follow the build instructions from here <https://github.com/xdp-project/xdp-tools> .

## Sources

Many sources have influenced this tutorial, including:

- <https://github.com/xdp-project/xdp-tutorial/>
- <https://developers.redhat.com/blog/2021/04/01/get-started-with-xdp>
