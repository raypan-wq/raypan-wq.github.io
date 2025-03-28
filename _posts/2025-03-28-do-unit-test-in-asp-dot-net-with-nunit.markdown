---
layout: post
title: "Do unit Test in ASP.Net with Nunit"
tags:
 - ASP.Net
 - unit test
---

Mark class as test container
```c#
[TestFixture]
[Parallelizable(ParallelScope.Fixtures)]
public class Basic {
```

Run before any test case
```c#
[OneTimeSetUp]
public void TestSetup() {
    string strPath = GetType().Namespace.Replace(".Testing", "").Replace("AI.FareCalculator.", "");
    _testCasePath = Path.Combine(TestContext.CurrentContext.WorkDirectory, @"..\..\AI.FareCalculator.Testing.Data", strPath, GetType().Name);
    _timeIntervaltoSave = new TimeSpan(0, 15, 0);
}
```

runs once after all tests `[OneTimeTearDown]`

The `[Values()]` attribute provides parameterized inputs
```c#
[Test, Order(10001)]
public void BasicURLTravelPattern001([Values(...)] string strsvcRequest) {
```

## Test Execution Flow
Discovery: NUnit scans the assembly for classes with [TestFixture] and methods with [Test], building a test suite.

Execution: Tests run in the specified order (via [Order]) or in parallel (per [Parallelizable]). For each test:

