---
title: Early strip() of an argument that can be a path or as-given URL loses data
date: "2026-09-20"
track: bug
category: runtime-errors
module: ds/launch.py
tags: [open, forward, path, whitespace, review]
problem_type: runtime-error
symptoms: file URL lost trailing whitespace; edge-whitespace file names resolved wrong or exited 2
root_cause: "one trimmed value served name lookup, URL forwarding and the filesystem test"
resolution_type: fix
---

## Problem
resolve_target trimmed its argument once at the top, which was harmless while every target was a name or an http(s) URL. Adding file:/about: forwarding and bare file paths made the trim lossy: a URL promised "as given" lost trailing whitespace, a file whose name starts or ends with whitespace resolved to a different file, a whitespace-only name returned early, and " file:page" was read as a URL after trimming.

## What Didn't Work
Forwarding raw.lstrip() and testing the path raw, while still detecting the scheme on the trimmed value: scheme detection and the value forwarded disagreed.

## Solution
ds/launch.py resolve_target: keep raw beside the trimmed arg. Forwarded schemes are detected on raw and forwarded as raw; the file test and quote(os.fsencode(...)) take raw; only name lookup and http(s) parsing read the trimmed value. Control characters are refused in a URL but allowed in a file name, where they percent-encode.

## Prevention
When an input gains a second meaning (name, now also path), audit every normalisation applied before the branch. Test with a decoy: create both " x " and "x" so a silent trim resolves to the wrong file and fails the URL assertion.
