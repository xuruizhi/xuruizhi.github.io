---
layout: post
title:  "Google在测试碎片化上的最新研究"
date:   2021-03-25 15:40:00 +0800
categories: "software_testing"
---

## Definition
- Flaky Tests，指在`被测对象`和`测试条件`都不变的情况下，有时候失败、有时候成功的测试，实际上就是不稳定的测试，或者随机失败（随机成功）的测试。
- Test Flakiness，指的是由于 `Flaky Tests` 的存在而导致的对测试效果的削弱，称之为`测试碎片化`。

> PS: 碎片化这个翻译，不太准确，但找不到更好的之前，姑且用之~

最新的 [Google Testing Blog](https://testing.googleblog.com/) 连续2期对测试碎片化进行了阐述，主要对Test Flakiness可能产生的原因进行分类并给出相应的消减措施。

- 2020.12: [Test Flakiness - One of the main challenges of automated testing](https://testing.googleblog.com/2020/12/test-flakiness-one-of-main-challenges.html)
- 2021.03: [Test Flakiness - One of the main challenges of automated testing (Part II)](https://testing.googleblog.com/2021/03/test-flakiness-one-of-main-challenges.html)

---

## Additional information

- 2020.07 Microsoft: [ICSE 2020: A Study on the Lifecycle of Flaky Tests](https://www.microsoft.com/en-us/research/publication/a-study-on-the-lifecycle-of-flaky-tests/)
- 2017.04 Google: [Where do our flaky tests come from](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)
- 2016.12 Google: [GTAC 2016: How Flaky Tests in Continuous Integration](https://www.youtube.com/watch?v=CrzpkF1-VsA)
- 2016.05 Google: [Flaky test at google and how we mitigate them](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html)

---

## References
[“投降论”和“速胜论”都不可取，消除 Flaky Tests 是一场持久战](https://mp.weixin.qq.com/s?__biz=MzI4NzczNjkxOQ==&mid=2247483868&idx=1&sn=142ba79e3ecfadd721c1cdec53641d2c&chksm=ebc854e4dcbfddf2)
