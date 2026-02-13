+++
title = "Generics Variance in Java"
date = 2025-01-19
description = "A comprehensive guide to understanding variance in Java generics: invariance, covariance, and contravariance."
+++

# Generics Variance in Java

Variance describes how subtyping between complex types (like `List<Dog>`) relates to subtyping between their component types (like `Dog` and `Animal`).

## Invariance (Default in Java)

By default, Java generics are **invariant** - no subtyping relationship exists between generic types, even if their type parameters have a subtyping relationship.

```java
class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}

// This does NOT compile - invariance
List<Animal> animals = new ArrayList<Dog>(); // ❌ Compile error
```

Even though `Dog` is a subtype of `Animal`, `List<Dog>` is NOT a subtype of `List<Animal>`. This prevents unsafe operations:

```java
// If the above were allowed, we could do:
animals.add(new Cat()); // Would corrupt a list of Dogs!
```

## Covariance (Producer - use `extends`)

**Covariance** allows reading from a generic type. Use `? extends T` when you only need to **get** values out.

```java
List<? extends Animal> animals = new ArrayList<Dog>(); // ✅ OK
Animal animal = animals.get(0); // ✅ Can read as Animal

animals.add(new Dog()); // ❌ Cannot add - compiler doesn't know exact type
animals.add(new Animal()); // ❌ Cannot add anything (except null)
```

**Use case - a method that processes any list of animals:**

```java
void printAnimals(List<? extends Animal> animals) {
    for (Animal animal : animals) {
        System.out.println(animal);
    }
}

printAnimals(new ArrayList<Dog>()); // ✅ Works
printAnimals(new ArrayList<Cat>()); // ✅ Works
```

## Contravariance (Consumer - use `super`)

**Contravariance** allows writing to a generic type. Use `? super T` when you only need to **put** values in.

```java
List<? super Dog> dogs = new ArrayList<Animal>(); // ✅ OK
dogs.add(new Dog()); // ✅ Can add Dog or its subtypes

Object obj = dogs.get(0); // ✅ Can only read as Object (not useful)
Dog dog = dogs.get(0); // ❌ Cannot read as Dog
```

**Use case - a method that adds dogs to any compatible list:**

```java
void addDogs(List<? super Dog> list) {
    list.add(new Dog());
    list.add(new Beagle()); // Beagle extends Dog
}

addDogs(new ArrayList<Dog>()); // ✅ Works
addDogs(new ArrayList<Animal>()); // ✅ Works
addDogs(new ArrayList<Object>()); // ✅ Works
```

## PECS Principle

**Producer Extends, Consumer Super** - a mnemonic for choosing variance:

- **Producer** (you read from it): use `? extends T`
- **Consumer** (you write to it): use `? super T`

```java
// Copy from source (producer) to dest (consumer)
<T> void copy(List<? extends T> source, List<? super T> dest) {
    for (T item : source) {
        dest.add(item);
    }
}

List<Dog> dogs = Arrays.asList(new Dog(), new Dog());
List<Animal> animals = new ArrayList<>();
copy(dogs, animals); // ✅ Works perfectly
```

## Real-World Example

```java
class AnimalShelter {
    // Covariant - accepts any list of animals to count
    int countAnimals(List<? extends Animal> animals) {
        return animals.size();
    }
    
    // Contravariant - adds rescued dogs to any compatible collection
    void rescueDogs(List<? super Dog> collection) {
        collection.add(new Dog());
        collection.add(new Dog());
    }
}

AnimalShelter shelter = new AnimalShelter();
shelter.countAnimals(new ArrayList<Dog>()); // ✅
shelter.countAnimals(new ArrayList<Cat>()); // ✅
shelter.rescueDogs(new ArrayList<Animal>()); // ✅
```

## Summary

- **Invariant**: No flexibility, exact type match required
- **Covariant** (`extends`): Read-only, flexible input
- **Contravariant** (`super`): Write-only, flexible output