---
title: Evaluating semgrep to find vulnerabilities in wordpress plugin
author: Tahu Datar
---
# Evaluating semgrep to find vulnerabilities in wordpress plugin


## Background

I wanted to learn SAST tools like semgrep or codeql...
...But, codeql doesn't support PHP language
...Since, I never used semgrep in the past...
The goal is to learn new ability using semgrep to enhance my code review skill. 
... to answer one simple question, **How effective a default semgrep rules to find vulnerability on wordpress plugins?**
In this article, I will try to tell my experience using semgrep to find vulnerabilities on wordpress plugins. 

## Part 1: Let's Do It

First of all, I don't have experience using semgrep in the past. So, the best way to learn it just running semgrep using rules that provided on official semgrep website.

During this experiment, I was used https://semgrep.dev/r?q=php.lang.security.file-inclusion.file-inclusion rule to find LFI on wordpress plugins.

I was running semgrep using this rule against 40.000+ wordpress plugins that downloaded to local machine. As a result, there was 800+ results that match to file inclusion rule.

## Validation Process

The most painful process during this experiment was validation process, I need to manually validate the results if its exploitable or not. Since I was used a default PHP file inclusion template, the rule was created to detect the simple source and sink. So, there was a lot of false positive during manual validation process. Within 800+ results, I was able to find only 27 valid LFI that reported to patchstack. 

But, during manual validation process, I was learn which code that really secure develop or not. Here is what I get:
1. Plugins that has secure protection always using allow whitelisting, which mean the code will check file before included to a function. For example, before the file is call by require, require_once, include, or include_once, it always check if file name is on allowed whitelist array or not. 
2. Using sanitizer doesn't mean the code is secure, sometimes I got plugins that using sanitizer but still can be bypassed. I always using this website https://qsandbox.com/tools/wp-functions/function/list to check if the sanitizer can be bypassed or not.


## Part 2: Do It Better

I wasn't satisfied with the default semgrep result, it still a lot of false positive results. So, I decided to create custom semgrep rule that can detect SQL injection. The idea is to detect input through the REST API as the source and wordpress SQL query function as the sink. To reduce false positive, I was add some sanitizer using wordpress function.   

