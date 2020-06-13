+++
title = "Mocking External Resources"
date = 2020-05-12T07:38:10+07:00
description = ""
draft = false
toc = false
categories = ["ppl"]
tags = [""]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image
+++

```
+------------------------------------------------------+
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                Beautiful Hero Image of               |
|                    Mock Objects                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
|                                                      |
+------------------------------------------------------+

```

Hello! Previously on [TDD][tdd-post] post we have briefly touched on the subject
of mocking. Django already provides tools to mock client request and database
connection. Do we need more?

<!--more-->

Obviously, the answer _depends_ on our circumstances. In cases where we need
it, we can count on python's `mock` module.

This is a simple example on how to use mock's `patch()` as a context manager in
a with statement.

```python
class Class:
    def method(self):
        pass


with patch('__main__.Class') as MockClass:
    instance = MockClass.return_value
    instance.method.return_value = 'foo'
    assert Class() is instance
    assert Class().method() == 'foo'
```

I prefer the with statement way because it enables clean and quick enter and
exit way to use the mocked object. But it's still a toy code though. Nothing
really happens there. Should we use mock there? Probably wasn't the best place
to use mock. But enough to give you simple idea how mock's side effect works.

One such example in my project is to mock the return values of anything
that depends on clock. This fits the *external resource* criteria as the clock
value is handled by the operating system. Other examples of external resource
includes network connection, file read, and vice versa.

```python
    def test_init_test_token(self):
        original_now = aware_utcnow()

        with patch('rest_framework_authlib.tokens.aware_utcnow') as fake_utc:
            fake_utc.return_value = original_now
            token = TestToken()

        token['test_key'] = 'test_value'
        encoded_token = str(token)

        now = aware_utcnow()

        with patch('rest_framework_authlib.tokens.aware_utcnow') as fake_utc:
            fake_utc.return_value = now
            t = TestToken(encoded_token)

        self.assertEqual(t.token, encoded_token)
        self.assertEqual(t.current_time, now)
```

Other examples where we use `patch()` as decorator and mock object instead of
return value.

```python
    @patch('rest_framework_authlib.authentication.AUTH_HEADER_TYPES',
           new=('JWT',))
    @patch('rest_framework_authlib.authentication.AUTH_HEADER_TYPE_BYTES',
           new=set((b'JWT',)))
    def test_get_raw_token_patched(self):
        # Should return None if header lacks correct type keyword
        self.assertIsNone(self.backend.get_raw_token(self.fake_header))
        # Should return None if an empty AUTHORIZATION header is sent
        self.assertIsNone(self.backend.get_raw_token(b''))
```

Now that we know a little about mocking objects and return values, dare I ask,
Can we mock all the things? This is an interesting question brought up on [this
stackexchange][mocking-just-right] post. How much mocking is just right for your
project? TL;DR, it depends on how much isolation level you need for your code
under tests. For our current project, only mocking external resources works. At
least for now.

[tdd-post]: /post/qa-using-sonarqube/
[mocking-just-right]: https://softwareengineering.stackexchange.com/questions/214516/how-much-mocking-is-just-right
