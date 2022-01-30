---
title: Preserving Manual Modifications to Auto-generated Code Across Versions
date: '2022-01-25'
author: Patrick Blesi
image: codegen_manual_edits.png
project_links:
  - title: Paper (TD Commons)
    url: https://www.tdcommons.org/dpubs_series/4862/
tags:
  - Code Generation
---

## Abstract

When an automatic code generator updates code that has been manually modified, the manual edits get overwritten and effectively lost. This is a problem for test and development engineers. In such situations, the user (test/development engineer) must laboriously re-apply the changes they made manually for each auto-generated update.

This disclosure describes techniques that preserve manual edits as auto-generated code undergoes automatic updates. The developer inserts sentinel values into the code, indicating lines that have been manually inserted or deleted. During automatic update, the sentinel values are used to recover the base file. An automatic code generator automatically generates a new revision of the file. The base file, the new revision, and the file with manual edits are merged using a three-way merge utility to obtain auto-updated code that includes the previously-made manual edits.
