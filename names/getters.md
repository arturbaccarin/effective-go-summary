# Getters

Go doesn't provide automatic support for getters and setters. There's nothing wrong with providing getters and setters yourself, and it's often appropriate to do so, but **_it's neither idiomatic nor necessary to put Get into the getter's name_**.

If you have a field called `owner` (lower case, unexported), **_the getter method should be called Owner_** (upper case, exported), **_not GetOwner_**. The use of upper-case names for export provides the hook to discriminate the field from the method. 

A **_setter function_**, if needed, will likely **_be called SetOwner_**.

```
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```