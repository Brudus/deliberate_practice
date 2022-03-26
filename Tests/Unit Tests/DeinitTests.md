# Test a method of deinit and check for memory leaks

## Introduction

I faced an issue at work where I wanted to test that a delegate unregisters from the sending object. While testing with a mock object, I noticed that even the delegate wasn't deallocated. 

## Solution

### What was causing the issue with the memory leak?

The memory leak was caused by the mock object I created. I stored the delegates as strong references in an array like this:

```
private(set) var delegates = [MyDelegate]()
```

That meant, even though I used the `sut = nil` in my tests, the instance was never deallocated and, therefore, never called its `deinit` method.

### How you can test for memory leaks?

Simply add a teardownBlock in your tests after the creation of the subject under test.

```
addTeardownBlock { [weak sut] in
    XCTAssertNil(sut, "Instance should have been deallocated. Potential memory leak.")
}
```

This will tell you if the instance was even deallocated.

> Pro tip: Use a helper method like `makeSUT()` to create your subject under test and always check if the instance was deallocated by adding the teardown block there. This adds checks for memory leaks to all your tests.

### How can you avoid this problem and test if the deinit method was called?

Instead of storing an array of delegates, you can store an array of closures with weak references to the delegates. 

```
private(set) var delegates = [() -> MyDelegate?]()
```

This will keep only weak references. Now your SUT will be deallocated correctly when setting to nil (if you have no other strong references).

Then you can simply add a check in the teardown block if the delegate was really removed. In my example, where the unregister method in the deinit should have removed the object, the teardown block looked like this:

```
addTeardownBlock { [weak sut] in
    XCTAssertNil(sut, "Instance should have been deallocated. Potential memory leak.")
    XCTAssertTrue(myMockService.delegates.isEmpty)
}
```

> I assume that this teardown block on the other hand holds now again a strong reference to the instance `myMockService`. In this case, however, this is intended behavior as I want to check if the delegate was really removed from the array (so the method to unregister on `myMockService` was really called).</br>An alternative would have been to simply store a variable if the unregister method was called. But I tend to avoid these flags as they easily make the mocks heavy and harder to use and reason about.