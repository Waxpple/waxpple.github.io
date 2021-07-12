---
title: "Day02"
date: 2021-07-12T10:01:41+08:00
draft: false
---

# Domains
A *domain* in its basic definition, is a grouping of logic cells. If consider a module as a blackbox. With its inputs and outputs, any given output is generated within one and only one domain. \
`Modules` come with two domain built in: a combinational domain and a sequential(synchronous) domain.
# Combinational
Logic that contains no clocked elements is called combinational logic. This is one of the domain that a `Module` contains. It is always named `comb`, and it can be accessed via m.d.comb. \
m means `Module` and d means `domain`.