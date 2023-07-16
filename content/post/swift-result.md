---
title: "Notes: Swift Result"
date: 2023-07-16T09:35:21+08:00
draft: false
tags: ["swift"]
toc: true
---

# Definition

```swift
/// A value that represents either a success or a failure, including an
/// associated value in each case.
@frozen public enum Result<Success, Failure> where Failure : Error {

    /// A success, storing a `Success` value.
    case success(Success)

    /// A failure, storing a `Failure` value.
    case failure(Failure)
}
```

# Example

```swift
struct Person: Decodable {
    let name: String
    let age: Int
}

extension String: Error {}

func loadPerson(from json: String) -> Result<Person, Error> {
    guard let data = json.data(using: .utf8) else {
        return .failure("Error in encoding string using utf8.")
    }
    do {
        let person = try JSONDecoder().decode(Person.self, from: data)
        return .success(person)
    } catch {
        return .failure(error)
    }
}

let json = """
{
    "name": "Jack Reacher",
    "age": 35
}
"""

let result = loadPerson(from: json)

switch result {
case .success(let person):
    print(person.name) // Jack Reacher
case .failure(let error):
    print(error.localizedDescription)
}
```

# Value Transformation

Its functions for transformaing associated sucess and failure values:
```swift
@inlinable public func map<NewSuccess>(_ transform: (Success) -> NewSuccess) -> Result<NewSuccess, Failure>

@inlinable public func mapError<NewFailure>(_ transform: (Failure) -> NewFailure) -> Result<Success, NewFailure> where NewFailure : Error

@inlinable public func flatMap<NewSuccess>(_ transform: (Success) -> Result<NewSuccess, Failure>) -> Result<NewSuccess, Failure>

@inlinable public func flatMapError<NewFailure>(_ transform: (Failure) -> Result<Success, NewFailure>) -> Result<Success, NewFailure> where NewFailure : Error
```

# Conversion between `Result` and throwing expression

## `Result` ➡️ throwing expression

The `get()` function of `Result` enables us to convert a `Result` value to a throwing expression.

```swift
@inlinable public func get() throws -> Success
```

```swift
let optionalPerson = try? loadPerson(from: json).get()
```

## Throwing expression ➡️ `Result`

Below is a throwing version of function `loadPerson(from:)`:
```swift
func throwingLoadPerson(from json: String) throws -> Person {
    guard let data = json.data(using: .utf8) else {
        throw "Error in encoding string using utf8."
    }
    let person = try JSONDecoder().decode(Person.self, from: data)
    return person
}
```

Wrap the throwing call as `Result`:

```swift
let personResult = Result { try throwingLoadPerson(from: json) }
```

The conversion is made possible by this function:

```swift
extension Result where Failure == Error {

    /// Creates a new result by evaluating a throwing closure, capturing the
    /// returned value as a success, or any thrown error as a failure.
    ///
    /// - Parameter body: A throwing closure to evaluate.
    public init(catching body: () throws -> Success)
}
```