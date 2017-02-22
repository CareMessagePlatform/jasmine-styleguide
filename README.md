# Jasmine Style Guide

## Purpose

The purpose of this style guide is to offer suggested best practices when writing Jasmine unit tests.

## Contributing

This style guide ultimately represents the opinions of its contributors. If you disagree with anything, or wish to add more, please create an issue or submit a pull request. Our goal is to continuously improve the guide and build consensus around it.

## Rules

1. [Speak Human](#speak-human)
1. [Write _Unit_ Tests](#write-unit-tests)
1. [Arrange-Act-Assert](#arrange-act-assert)
1. [Don't Repeat Yourself](#dont-repeat-yourself)
1. [`this` Is How We Do It](#this-is-how-we-do-it)
1. [Avoid the `All`s](#avoid-the-alls)
1. [Be `describe`tive](#be-describetive)
1. [Write Minimum Passable Tests](#write-minimum-passable-tests)

### Speak Human

Label your test suites (`describe` blocks) and specs (`it` blocks) in a way that clearly conveys the intention of each unit test. Note that the name of each test is the title of its `it` preceded by all its parent `describe` names. Favor assertive verbs and avoid ones like "should."

#### Why?

* Test suite becomes documentation for your codebase (helpful for new team members and non-technical stakeholders)
* Failure messages accurately depict what is broken
* Forces good naming conventions in tested code

#### Bad:

```javascript
// Output: "Array adds to the end"
describe('Array', function() {
  it('adds to the end', function() {
    var initialArray = [1];
    initialArray.push(2);
    expect(initialArray).toEqual([1, 2]);
  });
});
```

#### Better:

```javascript
// Output: "Array.prototype .push(x) appends x to the end of the Array"
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    it('appends x to the end of the Array', function() {
      var initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

### Write _Unit_ Tests

A unit test should test **one** thing. Confine your `it` blocks to a single assertion.

#### Why?

* Single responsibility principle
* A test can fail for only one reason

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    it('appends x to the end of the Array and returns it', function() {
      var initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    it('appends x to the end of the Array', function() {
      var initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      var initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
    });
  });
});
```

### Arrange-Act-Assert

Organize your code in a way that clearly conveys the 3 A's of each unit test. One way to accomplish this is by Arranging and Acting in `before` blocks and Asserting in `it` ones.

#### Why?

* The AAA unit test pattern is well known and recommended
* Improves unit test modularity and creates opportunities to DRY things up

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    it('appends x to the end of the Array', function() {
      var initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    var initialArray;

    beforeEach(function() {
      initialArray = [1]; // Arrange

      initialArray.push(2); // Act
    });

    it('appends x to the end of the Array', function() {
      expect(initialArray).toEqual([1, 2]); // Assert
    });
  });
});
```

### Don't Repeat Yourself

Use `before`/`after` blocks to DRY up repeated setup, teardown, and action code.

#### Why?

* Keeps test suite more concise and readable
* Changes only need to be made in one place
* Unit tests are not exempt from coding best practices

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    it('appends x to the end of the Array', function() {
      var initialArray = [1];
      initialArray.push(2);
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      var initialArray = [1];
      expect(initialArray.push(2)).toBe(2);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    var initialArray,
        pushResult;

    beforeEach(function() {
      initialArray = [1];

      pushResult = initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      expect(pushResult).toBe(2);
    });
  });
});
```

### `this` Is How We Do It

Use `this` to share variables between `it` and `before`/`after` blocks.

#### Why?

* Declare and initialize variables on one line
* Jasmine automatically cleans the `this` object between specs to avoid state leak

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    var initialArray,
        pushResult;

    beforeEach(function() {
      initialArray = [1];

      pushResult = initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      expect(pushResult).toBe(2);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    beforeEach(function() {
      this.initialArray = [1];

      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

### Avoid the `All`s

Prefer `beforeEach/afterEach` blocks over `beforeAll/afterAll` ones. The latter are not reset between tests.

#### Why?

* Avoids accidental state leak
* Enforces test independence
* Order of `All` block execution relative to `Each` ones is not always obvious

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    beforeAll(function() {
      this.initialArray = [1];
    });

    beforeEach(function() {
      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    beforeEach(function() {
      this.initialArray = [1];

      this.pushResult = this.initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(this.initialArray).toEqual([1, 2]);
    });

    it('returns x', function() {
      expect(this.pushResult).toBe(2);
    });
  });
});
```

### Be `describe`tive

Nest `describe` blocks liberally to create functional subsets.

#### Why?

* Allows tests to build on each other from least to most specific
* Creates tests that are easy to extend and/or refactor
* Makes branch testing easier and less repetitive
* Encapsulates tests based on their common denominator

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x) on an empty Array', function() {
    beforeEach(function() {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', function() {
      expect(this.initialArray).toEqual([1]);
    });
  });

  describe('.push(x) on a non-empty Array', function() {
    beforeEach(function() {
      this.initialArray = [1];

      this.initialArray.push(2);
    });

    it('appends x to the end of the Array', function() {
      expect(this.initialArray).toEqual([1, 2]);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    describe('on an empty Array', function() {
      beforeEach(function() {
        this.initialArray = [];

        this.initialArray.push(1);
      });

      it('appends x to the Array', function() {
        expect(this.initialArray).toEqual([1]);
      });
    });

    describe('on a non-empty Array', function() {
      beforeEach(function() {
        this.initialArray = [1];

        this.initialArray.push(2);
      });

      it('appends x to the end of the Array', function() {
        expect(this.initialArray).toEqual([1, 2]);
      });
    });
  });
});
```

### Write Minimum Passable Tests

If appropriate, use Jasmine's built-in matchers (such as `toContain`, `jasmine.any`, `jasmine.stringMatching`, ...etc) to compare arguments and results. You can also create your own matcher via the `asymmetricMatch` function.

#### Why?

* Tests become more resilient to future changes in the codebase
* Closer to testing behavior over implementation

#### Bad:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    beforeEach(function() {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', function() {
      expect(this.initialArray).toEqual([1]);
    });
  });
});
```

#### Better:

```javascript
describe('Array.prototype', function() {
  describe('.push(x)', function() {
    beforeEach(function() {
      this.initialArray = [];

      this.initialArray.push(1);
    });

    it('appends x to the Array', function() {
      expect(this.initialArray).toContain(1);
    });
  });
});
```
